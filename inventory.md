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

# `mbt-cache` - Global Inventory (GENERATED; DO NOT EDIT)

Generated: 2026-06-19T08:07:12Z
Protocol: code-only inventory; docs are excluded from source inventory.

This file is generated from per-component inventories under `crates/*/docs/inventory.md`, `services/*/docs/inventory.md`, and `benchmarks/*/docs/inventory.md`.
Nested crates under `crates/adapters/*` and `crates/schemas/*` are discovered by their `Cargo.toml` files.
Docs, target directories, and vendored dependency directories are excluded from source-file inventory.
If a file purpose is missing in a component inventory, this file will mark it as `INVENTORY GAP`.

## Components

- `crate::codegen`
- `crate::mbt-cache`
- `benchmark::mbt-cache-benchmarks`

---

## `crates/codegen`

### Source Files

- `crates/codegen/src/bin/mbt_cache_codegen.rs`: command-line entrypoint for cache codegen.
- `crates/codegen/src/cli.rs`: codegen CLI parsing and command dispatch.
- `crates/codegen/src/descriptor.rs`: protobuf descriptor and cache-option extraction.
- `crates/codegen/src/lib.rs`: codegen crate exports.
- `crates/codegen/src/model.rs`: descriptor-derived cache schema model.
- `crates/codegen/src/rust_emit.rs`: Rust adapter emitter for generated cache code.
- `crates/codegen/tests/test_codegen_check.rs`: generated-output check-mode test.
- `crates/codegen/tests/test_codegen_cli.rs`: codegen CLI behavior test.

---

## `crates/mbt-cache`

### Source Files

- `crates/mbt-cache/src/cache/api.rs`: public cache API and route dispatch surface.
- `crates/mbt-cache/src/cache/fresh_seed.rs`: fresh-build seed path and atomic publish boundary.
- `crates/mbt-cache/src/cache/mod.rs`: cache module exports.
- `crates/mbt-cache/src/cache/read_session.rs`: read-session routing over Heed and SQLite.
- `crates/mbt-cache/src/cache/recovery.rs`: persisted-root marker and recovery checks.
- `crates/mbt-cache/src/cache/state.rs`: persisted cache state and open-root metadata.
- `crates/mbt-cache/src/cache/write_batch.rs`: logical write batch over payload and lookup stores.
- `crates/mbt-cache/src/config.rs`: checked runtime configuration types.
- `crates/mbt-cache/src/error.rs`: runtime error and result types.
- `crates/mbt-cache/src/index/math.rs`: numeric helpers for predicate evaluation.
- `crates/mbt-cache/src/index/migration.rs`: SQLite lookup table creation and migration checks.
- `crates/mbt-cache/src/index/mod.rs`: index module exports.
- `crates/mbt-cache/src/index/query.rs`: SQLite predicate query planning and key-only result flow.
- `crates/mbt-cache/src/index/sqlite.rs`: SQLite connection, schema, and key-only lookup helpers.
- `crates/mbt-cache/src/lib.rs`: runtime crate exports.
- `crates/mbt-cache/src/payload/heed.rs`: Heed payload store over generated cache keys.
- `crates/mbt-cache/src/payload/key.rs`: generated cache-key byte handling.
- `crates/mbt-cache/src/payload/mod.rs`: payload module exports.
- `crates/mbt-cache/src/schema/mod.rs`: schema module exports.
- `crates/mbt-cache/src/schema/predicate.rs`: typed predicate model and validation contract.
- `crates/mbt-cache/src/schema/predicate_parser.rs`: predicate string parser.
- `crates/mbt-cache/src/schema/traits.rs`: schema traits implemented by generated adapters.
- `crates/mbt-cache/tests/generated/cache/bars_v1_cache.rs`: generated test cache adapter artifact; do not hand edit.
- `crates/mbt-cache/tests/generated/cache/mod.rs`: generated test cache module exports; do not hand edit.
- `crates/mbt-cache/tests/generated/mbt/bars_v1.rs`: generated test MBT schema artifact; do not hand edit.
- `crates/mbt-cache/tests/generated/mbt/mod.rs`: generated test MBT module exports; do not hand edit.
- `crates/mbt-cache/tests/generated/mod.rs`: generated test module exports; do not hand edit.
- `crates/mbt-cache/tests/test_config.rs`: runtime configuration tests.
- `crates/mbt-cache/tests/test_fresh_seed.rs`: fresh seed correctness and marker tests.
- `crates/mbt-cache/tests/test_payload_heed.rs`: Heed payload store tests.
- `crates/mbt-cache/tests/test_predicate_parser.rs`: predicate parser tests.
- `crates/mbt-cache/tests/test_read_routes.rs`: latest, range, search, and time-machine route tests.
- `crates/mbt-cache/tests/test_recovery.rs`: write journal and recovery tests.
- `crates/mbt-cache/tests/test_sqlite.rs`: SQLite schema and lookup tests.
- `crates/mbt-cache/tests/test_state_marker.rs`: state marker encode/decode tests.
- `crates/mbt-cache/tests/test_write_batch.rs`: point write and delete batch tests.

---

## `benchmarks/mbt-cache-benchmarks`

### Source Files

- `benchmarks/mbt-cache-benchmarks/src/benches/bars_parquet_seed.rs`: bars parquet seed benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/benches/bars_parquet_source.rs`: bars parquet source reader for seed benchmarks.
- `benchmarks/mbt-cache-benchmarks/src/benches/disk.rs`: benchmark disk-root lifecycle helpers.
- `benchmarks/mbt-cache-benchmarks/src/benches/latest_range_isolated.rs`: isolated latest/range route benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/benches/mod.rs`: benchmark module exports.
- `benchmarks/mbt-cache-benchmarks/src/benches/predicate_string_parser.rs`: predicate parser benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/benches/primitives_wide_row_comparison.rs`: primitives wide-row cache benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/benches/primitives_wide_row_source.rs`: primitives parquet source reader for wide-row benchmarks.
- `benchmarks/mbt-cache-benchmarks/src/benches/primitives_wide_row_workloads.rs`: primitives benchmark workload definitions.
- `benchmarks/mbt-cache-benchmarks/src/benches/search_time_machine_sqlite_key_only.rs`: search/time-machine SQLite key-only benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/benches/spool.rs`: benchmark spool and temporary artifact helpers.
- `benchmarks/mbt-cache-benchmarks/src/benches/sqlite_seed_optimization.rs`: SQLite seed optimization benchmark lane.
- `benchmarks/mbt-cache-benchmarks/src/dataset.rs`: shared benchmark dataset loading helpers.
- `benchmarks/mbt-cache-benchmarks/src/generated/cache/bars_v1_cache.rs`: generated artifact owned by cache codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/cache/mod.rs`: generated artifact owned by cache codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/cache/primitives_v1_cache.rs`: generated artifact owned by cache codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/cache/test_compatibility_v1_cache.rs`: generated artifact owned by cache codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/cache/wide_presence_v1_cache.rs`: generated artifact owned by cache codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mbt/bars_v1.rs`: generated artifact owned by MBT codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mbt/mod.rs`: generated artifact owned by MBT codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mbt/primitives_v1.rs`: generated artifact owned by MBT codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mbt/test_compatibility_v1.rs`: generated artifact owned by MBT codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mbt/wide_presence_v1.rs`: generated artifact owned by MBT codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/generated/mod.rs`: generated artifact owned by benchmark schema codegen; do not hand edit.
- `benchmarks/mbt-cache-benchmarks/src/main.rs`: benchmark command entrypoint and workload dispatch.
- `benchmarks/mbt-cache-benchmarks/src/report.rs`: benchmark report and evidence writing helpers.

---
