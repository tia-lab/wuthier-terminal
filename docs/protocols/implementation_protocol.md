```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Implementation

Version: 1.0
Status: active
Scope: code, generated artifacts, build files, and service adapters

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
- generated artifacts are bound when relevant,
- dependency changes are bound,
- tests and benchmarks are bound,
- failure contract is bound,
- correctness and privacy oracle is bound.

## Implementation Rules

- Implement only what the spec and plan bind.
- Prefer small, explicit modules.
- Keep Wuthier Terminal runtime small.
- Keep Agent Service adapters, Key Service adapters, OCR adapters, embedding
  adapters, database adapters, benches, fixtures, and generated test data
  outside production hot paths unless the spec proves otherwise.
- Validate trust-zone and sensitive-data boundaries before adapting, storing, or
  transmitting content.
- Keep hot paths allocation-conscious.
- Do not use `unwrap`, `expect`, or `panic!` in runtime, codegen, measurement,
  security-boundary, or reusable support logic.
- Do not add hidden defaults.
- Do not add unbound configuration.
- Do not add unbound dependencies.
- Do not widen the measured object.
- Do not create benchmark shortcuts that are not part of the measured system.
- Do not send plaintext sensitive values to Agent Service, RAG, embeddings,
  agent tools, LLM providers, logs, telemetry, or AI-facing stores.
- Do not expose Key Service routes to Agent Service or LLM tools.
- Do not store token dictionaries or decryption keys in AI-facing stores.
- Do not render plaintext without approved authorization and audit behavior.

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
- trust-zone routing matches spec,
- plaintext exclusion matches spec,
- authorization and audit behavior matches spec,
- failure behavior matches spec,
- correctness and privacy oracle can be tested,
- benchmark code measures the intended object,
- no unapproved behavior was added.

## Stop Gates

Stop implementation if:

- code requires behavior not in spec,
- dependency or service API contradicts spec assumptions,
- correctness or privacy oracle cannot be implemented,
- benchmark would measure a different object,
- hidden allocation or copy behavior cannot be bounded,
- code cannot stay within approved file bindings,
- generated output requires manual editing,
- plaintext sensitive values would reach AI-facing routes,
- Agent Service would need to call Key Service,
- token dictionaries or decryption keys would enter AI-facing stores,
- authorization or audit behavior cannot match the spec.
