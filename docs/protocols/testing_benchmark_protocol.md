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

# PROTOCOL: MBT Cache Testing and Benchmarking

Version: 1.0
Status: active
Scope: tests, benches, and evidence artifacts

## Purpose

Compilation is not proof. A benchmark number is not proof unless correctness,
determinism, and measurement boundaries are already established.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/implementation_protocol.md`
- `docs/protocols/code_style_protocol.md`
- approved spec
- approved implementation plan
- implemented code paths

## Entry Conditions

Testing may start only when:

- implementation is complete,
- pre-test audit is complete,
- test paths are bound,
- benchmark paths are bound,
- correctness oracle is implemented or ready to implement,
- artifact paths are bound.

## Test Categories

Every code change must cover relevant categories:

- Category A: contract and failure behavior.
- Category B: deterministic replay.
- Category C: correctness oracle.
- Category D: edge and corrupt input.
- Category E: generated-code reproducibility.
- Category F: Heed payload behavior when payload storage is involved.
- Category G: SQLite key-only lookup behavior when search or time-machine is
  involved.
- Category H: copy/allocation behavior when claimed.
- Category I: compile-surface behavior when generated code is touched.
- Category J: runtime benchmark execution when performance is claimed.
- Category K: atomic write, recovery, marker, and published-root behavior when
  persistence is touched.

## Benchmark Requirements

Benchmark artifacts must record:

- UTC timestamp,
- operator,
- git commit or dirty state,
- command,
- build profile,
- CPU,
- RAM,
- OS/kernel,
- Rust toolchain,
- input schema identity,
- Heed and SQLite identity when relevant,
- row count and payload size,
- warm/cold cache mode when relevant,
- result summary,
- raw output path.

## Compile Benchmark Requirements

Compile-surface artifacts must record:

- command,
- profile,
- clean/dirty state,
- target crate,
- enabled features,
- dependency graph condition,
- wall time,
- user/system CPU time when available,
- max RSS when available,
- generated source size,
- macro-expanded size when relevant.

## Performance Claim Rules

- Report median, tail, and throughput when the spec asks for latency.
- Separate seed, Heed write, SQLite lookup write, latest, range, search hit
  discovery, time-machine context replay, checked access, and trusted access
  when relevant.
- Do not compare two systems unless logical payload and correctness oracle are
  identical.
- Do not hide failed or unstable runs.
- Do not count benchmark-only spool files as production cache footprint unless
  the spec declares spool mode as the measured object.

## Failure Rule

When a test or benchmark fails, assume first that the implementation or
measurement design is wrong.

Do not change expected values, tolerances, or benchmark scope until independent
evidence proves the original expectation was wrong.

## Exit Conditions

Testing is complete only when:

- mandatory categories pass or blocked categories are explicitly justified,
- benchmark artifacts exist when performance is claimed,
- correctness oracle passes,
- determinism is demonstrated,
- result review is ready to write.
