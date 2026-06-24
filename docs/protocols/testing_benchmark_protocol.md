```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Testing and Benchmarking

Version: 1.0
Status: active
Scope: tests, benches, security-boundary checks, and evidence artifacts

## Purpose

Compilation is not proof. A benchmark number is not proof unless correctness,
privacy boundaries, determinism, and measurement boundaries are already
established.

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
- correctness and privacy oracle is implemented or ready to implement,
- artifact paths are bound.

## Test Categories

Every code change must cover relevant categories:

- Category A: contract and failure behavior.
- Category B: deterministic replay.
- Category C: correctness oracle.
- Category D: privacy oracle.
- Category E: edge, corrupt, stale, wrong-tenant, wrong-matter, and
  unauthorized input.
- Category F: generated-code reproducibility when generated code is touched.
- Category G: Agent Service plaintext-exclusion behavior when AI-facing routes
  are involved.
- Category H: Key Service isolation and authorization behavior when rendering
  or token dictionaries are involved.
- Category I: conversation, retrieval, embedding, log, and telemetry plaintext
  exclusion when persistence is involved.
- Category J: copy/allocation behavior when claimed.
- Category K: compile-surface behavior when generated code or dependency-heavy
  adapters are touched.
- Category L: runtime benchmark execution when performance is claimed.
- Category M: recovery, retry, cancellation, and degraded behavior when
  persistence or networking is touched.

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
- input data identity,
- storage identity when relevant,
- row, file, prompt, chunk, or token count when relevant,
- payload size when relevant,
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
- Separate file watch, extraction, detection, tokenization, redaction,
  embedding, retrieval, prompt tokenization, response rendering, and audit
  costs when relevant.
- Do not compare two systems unless logical payload and correctness/privacy
  oracle are identical.
- Do not hide failed or unstable runs.
- Do not count benchmark-only fixtures or spools as production footprint unless
  the spec declares them part of the measured object.

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
- privacy oracle passes,
- determinism is demonstrated,
- result review is ready to write.
