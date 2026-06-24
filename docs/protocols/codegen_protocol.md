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

# PROTOCOL: MBT Cache Codegen

Version: 1.0
Status: active
Scope: cache options, descriptor parsing, generator code, generated artifacts

## Purpose

Define how `.proto + MBT options + MBT Cache options` become generated Heed key
contracts, SQLite key-only lookup schemas, predicate binders, and read/write
adapter bindings.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/spec_protocol.md`
- `docs/protocols/implementation_protocol.md`
- approved spec
- approved implementation plan

## Source Contract

The codegen source of truth is:

```text
.proto files + approved MBT options + approved MBT Cache options
```

Schemas may live:

- inside this repository as fixtures;
- in a separate schema repository;
- inside an application crate.

The generator must receive explicit proto roots, proto file, root message, and
schema hash. It must not discover schema intent from Rust code.

## Generated Artifact Rules

1. Generated files are not edited by hand.
2. Generated files must include a deterministic header.
3. Generated files must be reproducible by a checked command.
4. Generated files must not include surfaces absent from the spec.
5. Generated code must reject unsupported schema and cache lookup shapes before
   runtime.
6. Generated code must keep checked and trusted MBT access distinct.
7. Generated code must not force unrelated schemas to compile.
8. Generated code must not include serde payload conversion.
9. Generated code must not write MBT payload bytes into SQLite.
10. Generated latest and range routes must target Heed directly.
11. Generated search and time-machine routes must return keys from SQLite and
    fetch payloads from Heed.

## Compile-Surface Rules

Generated-code specs must measure or bound:

- generated source lines,
- macro-expanded lines when relevant,
- `cargo check` time for affected crates,
- release build time when runtime performance code is added,
- dependency graph changes.

Wide schemas require explicit compile-surface evidence.

## Option Rules

MBT Cache options must be centralized and stable.

New options require a spec that defines:

- option name,
- protobuf extension target,
- allowed values,
- invalid combinations,
- descriptor behavior,
- generated runtime behavior,
- Heed key behavior when relevant,
- SQLite lookup behavior when relevant,
- query route behavior when relevant,
- test cases,
- compatibility risk.

## Stop Gates

Stop if:

- codegen output cannot be regenerated;
- generated output requires manual edits;
- an option has ambiguous behavior;
- a schema shape compiles but is unsupported at runtime;
- lookup behavior is inferred without an explicit option or proven fallback
  rule;
- codegen introduces schema-specific hardcoding not derived from proto;
- compile-surface impact is unknown for wide schemas.
