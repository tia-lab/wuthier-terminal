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

# MBT Cache Repository Structure

Status: initial architecture lock

## Purpose

This document records the intended structure for the production MBT Cache
repository. It is an architecture guardrail, not an implementation plan.

## Target Workspace Shape

```text
crates/mbt-cache
  package: mbt_cache
  runtime errors, cache config, Heed payload runtime, SQLite key-only lookup
  runtime, read/write session APIs, predicate runtime helpers, schema traits
  needed by generated code

crates/codegen
  package: mbt_cache_codegen
  protobuf descriptor reader, MATHILDE cache option validation, cache schema
  model derivation, generated Heed key and SQLite lookup adapter emitter,
  codegen CLI

benchmarks
  standalone benchmark crates and benchmark-only generated schemas, outside
  production runtime members when approved

proto
  MBT Cache-owned option definitions, if and when approved by spec

docs
  architecture, invariants, protocols, specs, reviews, benchmark evidence

bin
  repository maintenance tools only
```

The repository is named `mbt-cache`, so folder names stay short. Cargo package
names are explicit Rust import surfaces:

```text
mbt_cache
mbt_cache_codegen
```

## Ownership Model

MBT Cache uses MBT. It does not redefine MBT.

```text
MBT repository
  owns MBT wire/archive format
  owns MBT transport options
  owns mbt_core runtime

MBT Cache repository
  owns cache options and interpretation
  owns descriptor -> cache schema model
  owns generated Heed key contracts
  owns generated SQLite lookup schema and binders
  owns generated key-only search and time-machine hit discovery
  owns unified cache runtime API

Application or schema repository
  owns domain .proto schemas
  imports MBT and MBT Cache options
  runs approved codegen
```

## Runtime Flow

MBT payload bytes are the stored value and API boundary.

```text
validated MBT bytes
  -> generated key extraction
  -> Heed key/value payload write
  -> generated SQLite key-only lookup write
```

Read lanes:

```text
latest(entity)
  -> generated Heed key/range route
  -> borrowed MBT payload bytes

range(entity, start, end)
  -> generated Heed range route
  -> borrowed MBT payload bytes

search(predicate)
  -> generated SQLite predicate query
  -> key-only hits
  -> Heed payload replay
  -> borrowed MBT payload bytes

time_machine(predicate, before, after)
  -> same generated SQLite predicate query as search
  -> key-only hits
  -> deterministic Heed context lookup
  -> borrowed MBT payload bytes
```

Write lanes:

```text
checked MBT bytes
  -> checked MBT access
  -> generated key and predicate-column extraction
  -> Heed + SQLite atomic write

trusted MBT bytes
  -> unsafe trusted MBT access
  -> generated key and predicate-column extraction
  -> Heed + SQLite atomic write
```

SQLite must never store MBT payload bytes.

## Compile-Surface Rule

No crate should compile unrelated surfaces.

Examples:

- Runtime users of `mbt_cache` should not compile protobuf descriptor loading
  or emitter code.
- Codegen users of `mbt_cache_codegen` may compile descriptor tooling and
  formatting helpers.
- Benchmarks and wide test schemas must not be part of the default production
  runtime compile path.
- Application crates should compile only the generated schemas and cache
  adapters they import.
- Cache adapters must not force JSON, Arrow, Parquet, Postgres, or other MBT
  boundary adapters to compile.

## Schema Ownership

Schemas are `.proto` files and may live outside this repository.

Allowed ownership shapes:

```text
application crate
  proto/*.proto
  generated schema and cache adapter modules

schema repository
  proto/**/*.proto
  generated schema crate when approved

MBT Cache repository
  test schemas and compatibility fixtures only
```

MBT Cache options, when present, must be imported explicitly from the approved
option path. No Rust-only annotation is allowed to define cache behavior.

## Non-goals

- No monolithic crate that compiles runtime, codegen, every schema, and every
  benchmark together.
- No benchmark support inside production library crates.
- No generated code edited by hand.
- No schema-specific handwritten branches in generic infrastructure.
- No serde payload path in cache runtime.
- No old monolithic MBT dependency.
- No SQLite payload storage.
- No source-data finality, watermark, or hole-repair ownership.
- No performance or compile-time claim without recorded evidence.
