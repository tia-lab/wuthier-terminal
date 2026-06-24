```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Code Style

Version: 1.0
Status: active
Scope: Rust and support code

## Purpose

Define the minimum code discipline for Wuthier Terminal production and evidence
code.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- approved spec
- approved implementation plan
- `docs/protocols/implementation_protocol.md`
- `docs/protocols/testing_benchmark_protocol.md`

## Rules

1. Validation first.
2. Deterministic ordering.
3. Panic-free runtime, codegen, persistence, security-boundary, and measurement
   logic.
4. Explicit error handling.
5. Explicit copy and allocation boundaries.
6. No hidden defaults.
7. No unbound configuration.
8. No inline benchmark claims.
9. No implementation behavior absent from spec.
10. Small modules and explicit types.
11. No public API by accident.
12. No benchmark code in production libraries.
13. No plaintext sensitive values in AI-facing routes.
14. No Agent Service to Key Service route.
15. No token dictionary or decryption-key storage in AI-facing surfaces.
16. No plaintext rendering without approved authorization and audit behavior.

## Rust Shape

Preferred order inside non-trivial files:

1. imports,
2. constants,
3. config types,
4. error/result types,
5. public entrypoints,
6. validation helpers,
7. runtime, persistence, route, or adapter core,
8. artifact/report helpers,
9. tests only if the approved plan permits colocated unit tests.

`lib.rs` files should export modules and public items only. Move non-trivial
logic into named modules such as `client.rs`, `watch.rs`, `tokenize.rs`,
`redact.rs`, `agent.rs`, `key_service.rs`, `render.rs`, `audit.rs`, or
`storage.rs`, according to the approved spec.

## Error Handling

Runtime, persistence, codegen, security-boundary, and measurement logic must not
use:

- `unwrap`,
- `expect`,
- `panic!`,
- `todo!`,
- `unreachable!`,
- `unwrap_or` to hide failure,
- `unwrap_or_default` to hide failure.

Use typed errors or explicit result propagation.

## Sensitive-Data Discipline

When work touches plaintext, tokens, redaction, encryption, rendering, or
storage:

- identify all plaintext buffers;
- identify all tokenized and redacted buffers;
- identify all encrypted buffers;
- identify all borrowed views;
- identify all Human Zone, AI Zone, and Key Zone boundaries;
- keep copy points explicit;
- test plaintext exclusion when the spec requires it.

## Compile-Surface Discipline

Generated, adapter, and integration code must:

- avoid compiling unrelated service adapters;
- avoid compiling dependency-heavy tools into default runtime users;
- avoid compiling benchmark support into production libraries;
- avoid large generated functions unless the spec proves they are required;
- measure compile impact when wide schemas, generated code, or dependency-heavy
  adapters are involved.
