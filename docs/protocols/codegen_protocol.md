```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Codegen

Version: 1.0
Status: active
Scope: generated artifacts, schemas, contract bindings, and generated adapters

## Purpose

Define how approved source contracts become generated code or generated
artifacts when a spec permits generation.

This protocol does not approve any generator by itself. Generated tokenizers,
redactors, route adapters, schema bindings, test fixtures, or service clients
require a spec, peer audit, and approved implementation plan.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/spec_protocol.md`
- `docs/protocols/implementation_protocol.md`
- approved spec
- approved implementation plan

## Source Contract

The codegen source of truth must be explicit in the spec. Examples include:

- approved data schemas;
- approved service interface definitions;
- approved sensitive-data rule tables;
- approved redaction or tokenization rule sources;
- approved test-fixture definitions.

The generator must receive explicit input paths, output paths, root contracts,
and contract hashes when relevant. It must not discover security intent from
unstructured Rust code.

## Generated Artifact Rules

1. Generated files are not edited by hand.
2. Generated files must include a deterministic header.
3. Generated files must be reproducible by a checked command.
4. Generated files must not include surfaces absent from the spec.
5. Generated code must reject unsupported input combinations before runtime.
6. Generated code must keep checked and trusted boundaries distinct when both
   exist.
7. Generated code must not force unrelated adapters or schemas to compile.
8. Generated code must not send plaintext sensitive values to AI-facing routes.
9. Generated code must not expose Key Service routes to Agent Service or LLM
   tools.
10. Generated code must not store token dictionaries or decryption keys in
    AI-facing stores.
11. Generated code must preserve deterministic ordering when equality, replay,
    audit, or benchmarks depend on it.

## Compile-Surface Rules

Generated-code specs must measure or bound:

- generated source lines,
- macro-expanded lines when relevant,
- `cargo check` time for affected crates,
- release build time when runtime performance code is added,
- dependency graph changes.

Wide generated surfaces require explicit compile-surface evidence.

## Option and Rule Rules

New generated options or rule annotations require a spec that defines:

- option or rule name,
- source target,
- allowed values,
- invalid combinations,
- descriptor or parser behavior,
- generated runtime behavior,
- trust-zone behavior when relevant,
- storage behavior when relevant,
- test cases,
- compatibility risk.

## Stop Gates

Stop if:

- codegen output cannot be regenerated;
- generated output requires manual edits;
- an option or rule has ambiguous behavior;
- an input shape compiles but is unsupported at runtime;
- tokenization, redaction, or route behavior is inferred without an explicit
  rule or proven fallback;
- codegen introduces handwritten domain branches not derived from the approved
  source contract;
- compile-surface impact is unknown for wide generated surfaces.
