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

# PROTOCOL: MBT Cache Implementation

Version: 1.0
Status: active
Scope: code, generated artifacts, and build files

## Purpose

Control implementation after a spec and implementation plan are approved.

## Locked Refresh Rule

Before implementation, reread:

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/spec_protocol.md`
- `docs/protocols/code_style_protocol.md`
- `docs/protocols/codegen_protocol.md` when generated code is touched
- `docs/protocols/testing_benchmark_protocol.md`
- approved spec
- approved implementation plan

Memory of prior work is not evidence.

## Entry Conditions

Implementation may start only when:

- spec exists and is approved,
- peer audit passed,
- implementation plan exists and is approved,
- exact files are bound,
- generated artifacts are bound,
- dependency changes are bound,
- tests and benchmarks are bound,
- failure contract is bound,
- correctness oracle is bound.

## Implementation Rules

- Implement only what the spec and plan bind.
- Prefer small, explicit modules.
- Keep `mbt_cache` runtime small.
- Keep descriptor/codegen behavior in `mbt_cache_codegen`.
- Keep benches, fixtures, and generated test schemas outside production
  library code.
- Validate schema and cache metadata before adapting rows or opening persisted
  roots.
- Keep hot paths allocation-conscious.
- Do not use `unwrap`, `expect`, or `panic!` in runtime, codegen, measurement,
  or reusable support logic.
- Do not add hidden defaults.
- Do not add unbound configuration.
- Do not add unbound dependencies.
- Do not widen the measured object.
- Do not create benchmark shortcuts that are not part of the measured system.
- Do not reintroduce the old monolithic MBT dependency.
- Do not add a serde payload path.
- Do not store MBT payload bytes in SQLite.
- Do not route latest or range through SQLite.
- Do not make time-machine context depend on SQLite payload data.

## Pre-Test Audit

Before testing, audit final code against:

- approved spec,
- approved implementation plan,
- `code_style_protocol.md`,
- `codegen_protocol.md` when relevant,
- expected test and benchmark bindings.

The audit must confirm:

- code path matches binding,
- generated files match codegen output,
- dependency changes match binding,
- failure behavior matches spec,
- correctness oracle can be tested,
- benchmark code measures the intended object,
- no unapproved behavior was added.

## Stop Gates

Stop implementation if:

- code requires behavior not in spec,
- dependency API contradicts spec assumptions,
- correctness oracle cannot be implemented,
- benchmark would measure a different object,
- hidden allocation or copy behavior cannot be bounded,
- code cannot stay within approved file bindings,
- generated output requires manual editing,
- Heed and SQLite cannot be updated as one logical write unit,
- SQLite would need to store MBT payload bytes,
- query routing would violate latest/range or search/time-machine boundaries.
