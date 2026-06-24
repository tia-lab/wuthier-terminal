# mbt-cache architecture

This document explains the internal architecture of `mbt-cache`. It is not
benchmark evidence. Runtime, compile-time, seed, and query-speed claims remain
bound to result reviews and benchmark artifacts.

## Scope

`mbt-cache` is a schema-generated local cache database for MBT payloads.

It owns:

- a Heed payload store for MBT bytes;
- a SQLite lookup store for key-only predicate hit discovery;
- one public cache API over both stores;
- schema-specific generated cache adapters;
- benchmark-only crates for evidence.

It does not own feed finality, watermark policy, repair policy, source-data
semantics, or the MBT wire format.

## Crate Map

```text
+-------------------------- REPOSITORY MAP --------------------------+
|                                                                    |
|  crates/mbt-cache                                                  |
|      src/cache/api.rs                                              |
|          public CacheDb API and request types                      |
|      src/cache/read_session.rs                                     |
|          payload-only latest/range and indexed search/time-machine |
|      src/cache/cursor.rs                                           |
|          opaque route cursor parsing, signatures, and encoding     |
|      src/cache/fresh_seed.rs                                       |
|          empty-root seed and atomic publish flow                   |
|      src/cache/write_batch.rs                                      |
|          point write/delete commit logic                           |
|      src/cache/recovery.rs                                         |
|          Heed/SQLite marker comparison and journal recovery        |
|      src/cache/state.rs                                            |
|          schema generation marker and payload checksum             |
|      src/payload/heed.rs                                           |
|          Heed key/value payload store                              |
|      src/index/sqlite.rs                                           |
|          SQLite open modes and connection setup                    |
|      src/index/migration.rs                                        |
|          internal/generated SQLite schema execution                |
|      src/index/math.rs                                             |
|          deterministic predicate math functions                    |
|      src/schema/traits.rs                                          |
|          generated adapter runtime contract                        |
|      src/schema/predicate*.rs                                      |
|          typed predicate AST and text parser                       |
|                                                                    |
|  crates/codegen                                                    |
|      descriptor model and generated cache adapter emitter          |
|                                                                    |
|  benchmarks/mbt-cache-benchmarks                                   |
|      benchmark-only schemas, datasets, harnesses, and evidence     |
|                                                                    |
+--------------------------------------------------------------------+
```

Runtime code does not load descriptors. Codegen code does not participate in
serving hot paths. Benchmark code is outside the production runtime crate.

## Storage Model

```text
+--------------------------- STORAGE MODEL --------------------------+
|                                                                    |
|  MBT row bytes                                                     |
|       |                                                            |
|       v                                                            |
|  generated CacheSchema adapter                                     |
|       |                                                            |
|       +--> sortable key                                            |
|       |        |                                                   |
|       |        v                                                   |
|       |   Heed payload store                                       |
|       |        key -> borrowed MBT bytes                           |
|       |                                                            |
|       +--> lookup columns                                          |
|                |                                                   |
|                v                                                   |
|           SQLite lookup store                                      |
|                key columns + predicate columns + state/journal     |
|                                                                    |
+--------------------------------------------------------------------+
```

Heed is the only MBT payload store. SQLite never stores MBT payload bytes.
SQLite returns keys for predicate routes; Heed returns payload slices.

The same generated key contract is used by seed, writes, latest, range,
search, and time-machine.

## Open Path

```text
+----------------------------- OPEN PATH ----------------------------+
|                                                                    |
|  CacheDb::<S>::open(root, config)                                  |
|       |                                                            |
|       +--> PayloadStore::open(root/heed, map_size)                 |
|       |                                                            |
|       +--> open_connection(root/sqlite/cache.sqlite)               |
|       |        -> WAL + FULL synchronous                           |
|       |        -> temp_store MEMORY                                |
|       |        -> deterministic math functions                     |
|       |        -> internal SQLite state tables                     |
|       |                                                            |
|       +--> create_generated_schema(S::SQLITE_DDL)                  |
|       |                                                            |
|       +--> ensure_initial_markers()                                |
|                                                                    |
+--------------------------------------------------------------------+
```

Operational opens configure SQLite for durable point mutations. Fresh seed uses
a separate open mode described below.

## State Markers

Both stores hold the same `StateMarker` for the schema:

- `schema_id`;
- `schema_hash`;
- `cache_schema_hash`;
- generation;
- payload key count;
- payload byte count;
- payload checksum.

Indexed reads compare Heed and SQLite markers before serving. Fresh seed
validates markers, row counts, and route smoke checks before publish. Point
writes advance both markers through one journaled generation transition.

## Fresh Seed

```text
+----------------------------- FRESH SEED ---------------------------+
|                                                                    |
|  CacheDb::<S>::build_fresh_checked/trusted(final_root, config, src)|
|       |                                                            |
|       +--> create private temporary root                           |
|       |                                                            |
|       +--> open_fresh_seed(temp_root)                              |
|       |        -> SQLite journal_mode OFF                          |
|       |        -> SQLite synchronous OFF                           |
|       |        -> empty Heed + empty SQLite markers                |
|       |                                                            |
|       +--> source.replay_seed(bytes)                               |
|       |        -> generated checked/trusted seed inserter          |
|       |        -> generated key validation                         |
|       |        -> SQLite lookup row insert                         |
|       |        -> Heed key -> MBT payload write                    |
|       |        -> marker contribution                              |
|       |                                                            |
|       +--> write matching Heed and SQLite markers                  |
|       |                                                            |
|       +--> validate temp root                                      |
|       |        -> SQLite quick_check                               |
|       |        -> marker comparison                                |
|       |        -> lookup row count                                 |
|       |        -> generated route smoke                            |
|       |                                                            |
|       +--> rename temp root to final root                          |
|       |                                                            |
|       +--> reopen final root and validate again                    |
|                                                                    |
+--------------------------------------------------------------------+
```

`seed_checked` and `seed_trusted` seed an already opened empty root. The
`build_fresh_*` APIs build a rebuildable artifact in a temporary root and
publish it only after validation.

Checked seed validates MBT bytes through the generated schema boundary.
Trusted seed is an explicit caller contract that the bytes were already
validated for the same schema before entering the seed path.

## Point Writes

```text
+---------------------------- POINT WRITES --------------------------+
|                                                                    |
|  insert / upsert / update / delete                                 |
|       |                                                            |
|       +--> writer mutex                                            |
|       |                                                            |
|       +--> recover_or_fail()                                       |
|       |                                                            |
|       +--> read Heed marker and SQLite marker                      |
|       |        -> markers must match                               |
|       |                                                            |
|       +--> stage SQLite journal rows                               |
|       |        -> operation                                        |
|       |        -> generation                                       |
|       |        -> key                                              |
|       |        -> old/new row contributions                        |
|       |                                                            |
|       +--> mutate Heed payloads and Heed marker                    |
|       |                                                            |
|       +--> mutate SQLite lookup rows and SQLite marker             |
|       |                                                            |
|       +--> clear journal                                           |
|                                                                    |
+--------------------------------------------------------------------+
```

Point writes validate MBT bytes before deriving keys. Deletes take a generated
key buffer. The writer mutex serializes mutation of the two-store state.

Recovery supports the documented one-generation transition table:

- no journal and matching markers: serve;
- journal with both stores old: clear journal;
- journal with both stores new: clear journal;
- Heed new and SQLite old: roll SQLite forward from the journal;
- SQLite new and Heed old: unrecoverable;
- any other marker shape: unrecoverable.

## Latest Route

```text
+----------------------------- LATEST -------------------------------+
|                                                                    |
|  CacheDb::<S>::latest(request, max_response_bytes, sink)           |
|       |                                                            |
|       +--> payload_read_session(ReadMode::Trusted)                 |
|       |        -> open Heed read transaction                       |
|       |        -> require Heed marker                              |
|       |        -> no SQLite transaction                            |
|       |                                                            |
|       +--> validate latest time grid and safe bound                |
|       |                                                            |
|       +--> S::for_each_route_entity(filter)                        |
|       |                                                            |
|       +--> S::write_key(entity, max_closed_time, key)              |
|       |                                                            |
|       +--> Heed get_payload(key)                                   |
|       |                                                            |
|       +--> sink.accept_latest_mbt(key view, borrowed payload)      |
|                                                                    |
+--------------------------------------------------------------------+
```

Latest is a payload-only route. It never opens or queries SQLite. The public
route uses trusted access because it serves bytes from a previously validated
cache root.

## Range Route

```text
+------------------------------ RANGE -------------------------------+
|                                                                    |
|  CacheDb::<S>::range(request, max_response_bytes, sink)            |
|       |                                                            |
|       +--> payload_read_session(ReadMode::Trusted)                 |
|       |        -> open Heed read transaction                       |
|       |        -> require Heed marker                              |
|       |        -> no SQLite transaction                            |
|       |                                                            |
|       +--> validate start/end/safe time grid                       |
|       |                                                            |
|       +--> entity_filter One                                      |
|       |        -> S::write_range_bounds(entity, start, end)        |
|       |        -> Heed ordered range scan                          |
|       |        -> enforce every expected key                       |
|       |                                                            |
|       +--> entity_filter Many                                     |
|                -> validate duplicate-free entity list              |
|                -> time-major generated point keys                  |
|                -> Heed get_payload(key) per point                  |
|                                                                    |
+--------------------------------------------------------------------+
```

One-entity ranges use ordered Heed scans. Multi-entity ranges use generated
point keys to preserve deterministic time-major output.

## Search Route

```text
+------------------------------ SEARCH ------------------------------+
|                                                                    |
|  CacheDb::<S>::read_session(mode).search_into(request, max, sink)  |
|       |                                                            |
|       +--> open Heed read transaction                              |
|       |                                                            |
|       +--> open SQLite transaction                                 |
|       |                                                            |
|       +--> compare Heed marker with SQLite marker                  |
|       |                                                            |
|       +--> validate request time grid and safe bound               |
|       |                                                            |
|       +--> S::preflight_search_request(request)                    |
|       |        -> schema-generated predicate/entity validation     |
|       |                                                            |
|       +--> S::search_hits_into(sqlite_txn, request, consume)       |
|       |        -> generated SQLite query over lookup columns       |
|       |        -> SQLite emits key-only CacheRouteHit              |
|       |                                                            |
|       +--> Heed get_payload(hit.key)                               |
|       |                                                            |
|       +--> optional audit validation                               |
|       |                                                            |
|       +--> sink.accept_search_mbt(key view, borrowed payload)      |
|                                                                    |
+--------------------------------------------------------------------+
```

SQLite is only the hit-discovery surface. If SQLite returns a key that Heed
does not have, the route fails with a stale lookup row error.

## Time-Machine Route

```text
+--------------------------- TIME-MACHINE ---------------------------+
|                                                                    |
|  predicate source                                                  |
|       -> same generated hit discovery as search                    |
|       -> for each hit, compute before/after time window            |
|       -> generated Heed point keys                                 |
|       -> Heed get_payload(key) for each context row                |
|       -> sink.accept_time_machine_mbt(...)                         |
|                                                                    |
|  explicit hit source                                               |
|       -> skip SQLite hit query                                     |
|       -> use generated explicit-hit entity/time extraction         |
|       -> same Heed context replay                                  |
|                                                                    |
+--------------------------------------------------------------------+
```

Time-machine uses the search engine only to discover hit keys when the source
is a predicate. Explicit hits still use an indexed read session for marker
parity, but they do not query SQLite for hits. Context replay is Heed-only. The
current partial-window policy is `Reject`; missing context rows fail the
request.

## Pagination Cursor Routes

```text
+------------------------- PAGINATION CURSORS -----------------------+
|                                                                    |
|  range_page                                                        |
|       -> payload-only Heed session                                 |
|       -> validate optional cursor before scan/get work             |
|       -> emit borrowed MBT payloads                                |
|       -> return optional CacheRouteStats.next_cursor               |
|                                                                    |
|  search_page                                                       |
|       -> indexed SQLite + Heed session                             |
|       -> validate optional cursor before SQLite query              |
|       -> generated SQLite key-only hit page                        |
|       -> Heed get_payload(hit.key)                                 |
|       -> return optional CacheRouteStats.next_cursor               |
|                                                                    |
|  time_machine_page                                                 |
|       -> same generated predicate hit page as search_page          |
|       -> Heed context replay around each hit                       |
|       -> return optional CacheRouteStats.next_cursor               |
|                                                                    |
+--------------------------------------------------------------------+
```

Pagination is route metadata, not payload data. `next_cursor` is returned in
`CacheRouteStats`; MBT bytes, projections, JSON, protobuf, and other adapter
outputs are not modified by pagination.

The cursor string is opaque to callers. Internally it is:

```text
v1|route|request_signature|generation|resolved_time_end|window_start|window_end|after_close_ms|after_entity_ordinal
```

The request signature binds:

- cursor version and route;
- generated `SCHEMA_ID`, `SCHEMA_HASH`, and `CACHE_SCHEMA_HASH`;
- cache generation;
- request time window;
- page limit;
- entity filter;
- predicate shape for search and predicate time-machine;
- `before`, `after`, and partial-window policy for time-machine pages.

Cursor validation happens before route storage access. Route mismatch, request
signature mismatch, generation mismatch, invalid time grid, and window mismatch
fail without reading payload rows.

Range cursors store the last emitted entity ordinal. Search and time-machine
cursors use a generated timestamp-complete sentinel because their SQLite hit
queries page by hit timestamp.

Latest is not cursor-paged. Explicit-hit time-machine is not cursor-paged
because the caller owns the hit list. Non-paged routes keep their existing
response shape.

Cursor encoding allocates only the returned `String`. Numeric fields are
written directly into that final string; payload bytes are not copied. SQLite
predicate routes still allocate request-level SQL and bind storage as part of
query construction.

Cursor route support comes from `mathilde.cache_route.cursor` in the schema
proto. The generated adapter decides whether range, search, and time-machine
cursor routes are available for that schema.

## Generated Adapter Contract

Generated adapters implement `CacheSchema`. Runtime code stays
schema-generic; schema-specific layout is emitted from proto options.

The generated adapter owns:

- schema identity constants:
  - `SCHEMA_ID`;
  - `SCHEMA_HASH`;
  - `CACHE_SCHEMA_HASH`;
  - `KEY_LEN`;
  - `TIME_STEP_MS`;
- entity contract:
  - entity list;
  - entity ordinal;
  - entity validation;
  - route entity iteration;
- key contract:
  - `write_key`;
  - `write_key_time`;
  - `write_range_bounds`;
  - `write_key_from_mbt`;
  - `write_key_from_sqlite_row`;
  - `explicit_hit_from_key`;
- SQLite contract:
  - `SQLITE_DDL`;
  - lookup row insert/delete;
  - seed lookup inserter;
  - row count;
  - journal clear/status/stage/recovery;
- predicate route contract:
  - preflight validation;
  - key-only search hit emission;
  - key-only search page emission;
  - text predicate field/entity parsing when enabled;
  - cursor entity-filter signature;
  - predicate cursor signature;
- validation contract:
  - MBT validation via `MbtSchema::access_view`;
  - fresh-seed route smoke.

Generated files are command-owned and are not hand edited.

## Predicate Architecture

```text
+---------------------------- PREDICATES ----------------------------+
|                                                                    |
|  typed predicate or predicate string                               |
|       |                                                            |
|       v                                                            |
|  CachePredicate AST                                                |
|       |                                                            |
|       v                                                            |
|  generated schema field/entity validation                          |
|       |                                                            |
|       v                                                            |
|  generated SQLite WHERE expression                                 |
|       |                                                            |
|       v                                                            |
|  key-only hits                                                     |
|                                                                    |
+--------------------------------------------------------------------+
```

The string parser is a convenience boundary. It is bounded before storage
access and supports schema-known entity-qualified fields, comparison, boolean
composition, arithmetic, `sqrt`, and `pow`.

SQLite math functions are deterministic and registered at connection open.
Invalid fields, entities, unsupported predicates, unsafe time bounds, and
non-finite math fail before payload replay.

## Checked and Trusted Access

Checked access:

- validates MBT bytes before deriving a key;
- is used by point writes;
- is available for seed and read-session audit mode.

Trusted access:

- is an explicit caller contract;
- is used by public latest/range routes;
- is available for fresh seed and read sessions when the caller has already
  proved bytes at the correct schema boundary;
- does not change storage format.

Trusted mode does not mean unchecked cache state. Marker comparison, key
validation, response-size caps, time-grid validation, and route failure checks
remain active.

## Failure Surfaces

Important failure surfaces are explicit:

- missing Heed marker;
- missing SQLite marker for indexed sessions;
- Heed/SQLite marker mismatch;
- unrecoverable journal state;
- invalid generated key length;
- invalid time grid;
- unsafe request bound;
- duplicate entity in `Many`;
- missing latest payload;
- range hole;
- partial time-machine window;
- stale SQLite lookup row;
- invalid predicate syntax;
- unsupported predicate or schema route;
- predicate math domain error;
- response too large;
- fresh seed non-empty root or empty source.

## Benchmark Boundary

```text
+-------------------------- BENCHMARK BOUNDARY ----------------------+
|                                                                    |
|  benchmarks/mbt-cache-benchmarks                                   |
|       -> benchmark-only generated MBT modules                      |
|       -> benchmark-only generated cache adapters                   |
|       -> dataset loaders                                           |
|       -> evidence JSON and markdown summaries                      |
|                                                                    |
|  crates/mbt-cache                                                  |
|       -> production runtime only                                   |
|                                                                    |
+--------------------------------------------------------------------+
```

Architecture diagrams explain intended data movement. They are not performance
claims. Performance claims require benchmark result files with command,
dataset, profile, and output evidence.

## Non-Goals

- No source-data finality guarantee.
- No predictive semantics.
- No general SQL query API.
- No SQLite payload storage.
- No handwritten schema-specific runtime branches.
- No benchmark fixtures in the runtime crate.
- No replacement of MBT schema validation.
