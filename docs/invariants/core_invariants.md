```
MATHILDE PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 MATHILDE. All Rights Reserved.

This document contains trade secrets and confidential information owned
exclusively by MATHILDE, protected under Swiss law (URG, UWG, Art. 162 StGB).

PROHIBITED: Reproduction, copying, distribution, disclosure, or derivative
works without prior written authorization from MATHILDE.

ACCESS REQUIREMENT: Executed NDA with MATHILDE required. Unauthorized access
or possession violates Swiss law. Violations subject to civil remedies,
injunctive relief, damages, and criminal prosecution.

Legal Contact: massimo.nicora@wnlegal.ch
```

# MBT Cache Core Invariants

Status: active
Scope: entire repository

These invariants are mandatory unless a later approved spec explicitly narrows
or supersedes one. Supersession must be local, justified, and audited.

## Schema and Storage Invariants

1. `.proto + MBT options + MBT Cache options` is the source of truth.
2. MBT Cache does not define the MBT wire/archive format.
3. Heed stores MBT payload bytes by generated cache key.
4. SQLite stores generated key-only lookup rows and searchable predicate
   columns.
5. SQLite must never store MBT payload bytes.
6. Latest and range routes must read payload bytes directly from Heed.
7. Search and time-machine routes must use SQLite only for key discovery.
8. Time-machine context must be built from Heed by deterministic key/range
   logic, not from SQLite payloads.
9. Application schemas may live outside this repository.
10. Generated schema and adapter code must come only from approved codegen.
11. Generated files are not edited by hand.

## Crate Boundary Invariants

1. `mbt_cache` is the runtime crate.
2. `mbt_cache_codegen` is the descriptor/codegen crate.
3. Codegen is a tool/build/dev surface, not the default runtime surface.
4. Descriptor dependencies such as `prost-reflect` must not enter
   `mbt_cache` unless a spec proves the need.
5. Benchmarks, fixtures, and wide test schemas must not be part of the default
   production runtime compile path.
6. Runtime users must not compile unrelated generated schemas.
7. Runtime users must not compile unrelated MBT boundary adapters.
8. Features may be used for narrow optional behavior, but they are not the main
   isolation mechanism for generated schemas or benches.

## Codegen Invariants

1. Codegen output must be deterministic.
2. Codegen input must include exact proto roots, proto file, root message, and
   schema hash.
3. Codegen must reject ambiguous cache annotations.
4. Codegen must reject unsupported field/type combinations before runtime.
5. Codegen must not emit handwritten schema-specific branches.
6. Codegen must not infer schema intent from Rust code.
7. Codegen must emit checked and trusted adapter surfaces as distinct APIs.
8. Unsafe generated APIs require a documented safety contract.
9. Codegen must keep compile surface bounded and measured for wide schemas.
10. Generated adapters must be reproducible by a checked command.

## Runtime Invariants

1. Inserts, upserts, deletes, and batch seeds must update Heed and SQLite as one
   logical unit.
2. Published fresh cache roots must be reopenable and marker-consistent.
3. Heed and SQLite row counts must match after validated seed/build flows.
4. Markers must detect incomplete or mismatched payload/index state.
5. Search preflight must reject unknown entities, fields, parameters, and
   unsupported predicates before storage access.
6. Query output ordering must be deterministic.
7. Safe serving semantics must come from the source system or application
   contract; MBT Cache must not invent finality.

## Adapter Invariants

1. Cache adapters consume and return MBT bytes, not serde DTO payloads.
2. No serde payload path is allowed in runtime adapters.
3. Optional values must preserve MBT presence semantics.
4. Checked MBT lanes validate/access before writing cache surfaces.
5. Trusted MBT lanes are unsafe and allowed only for already validated
   immutable bytes.
6. Hot adapter paths must not allocate or copy unless the spec declares the
   copy point.
7. All row ordering used for equality or benchmarks must be deterministic.
8. Failure behavior must be explicit for corrupt bytes, wrong schema, old
   version, missing payloads, missing lookup rows, and stale indexes.

## Benchmark Invariants

1. Performance claims require run evidence.
2. Compile-time claims require build evidence.
3. Correctness and determinism must pass before speed claims.
4. Baselines must use identical logical payloads.
5. Warm-cache and cold-cache results must be separated when storage is involved.
6. Failed and unstable runs are evidence and must not be hidden.
7. Tolerances must not be relaxed before proving correctness.
8. Benchmark setup work must be outside the measured loop unless the spec
   declares it part of the measured object.
9. Benchmark-only spool, parquet, or generated fixture surfaces must not become
   production runtime requirements.

## Documentation Invariants

1. Specs are the source of truth.
2. Peer audits are separate artifacts.
3. Implementation plans are separate artifacts.
4. Result reviews record evidence and limitations.
5. Documentation must separate proved facts from hypotheses.
6. Conversation trace is not documentation.
7. Public claims must not exceed recorded evidence.
8. A first spec draft must already close prior-spec conflicts, command
   surfaces, generated-artifact ownership, dispatch path bindings, exact code
   bindings, and test migration from old behavior to new behavior.
9. A peer audit must not be used as a substitute for author-side spec
   completeness. If basic protocol compliance is missing, the work restarts at
   spec authoring.
