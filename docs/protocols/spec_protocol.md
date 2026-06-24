```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Spec Authoring

Version: 1.0
Status: active
Scope: `docs/specs/*_SPEC.md`

## Purpose

The spec is the source of truth. Implementation may not contain a behavior,
dependency, file binding, generated artifact, benchmark, security claim, or
interpretation not present in the spec.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/research_protocol.md`
- this protocol
- target research brief
- relevant source code
- relevant service or dependency docs

## Entry Conditions

Spec authoring may start only when:

- slug is known,
- measured object is stated,
- candidate approach is known,
- major unknowns are listed,
- no code change is required to finish the spec.

## Mandatory Spec Location

- `docs/specs/[slug]_SPEC.md`

## Mandatory Sections

Use this section order:

1. Identification
2. Status
3. Purpose
4. Non-goals
5. Measured object
6. Source data contract
7. Sensitive-data classification contract
8. Tokenization contract
9. Redaction contract
10. Trust-zone contract
11. Agent Service boundary contract
12. Key Service boundary contract
13. Storage contract
14. Prompt and conversation contract
15. Rendering authorization and audit contract
16. Codegen or generated-artifact contract
17. Crate and service boundary contract
18. Dependency contract
19. Determinism contract
20. Failure contract
21. Compile-surface budget
22. Runtime performance budget
23. Correctness and privacy oracle
24. Benchmark methodology
25. Test plan
26. Code bindings
27. Generated and evidence artifact bindings
28. Review artifact bindings
29. Implementation plan requirement
30. Approval checklist
31. Open questions

## Pre-Audit Closure Gate

Before requesting or writing the first peer audit, the spec author must prove
the spec is internally closed. This gate is inviolable.

The first spec draft must explicitly verify:

- mandatory section order matches this protocol exactly;
- every prior approved spec that may conflict has been searched and either
  preserved, superseded locally, or cited as out of scope;
- every command surface is exact, including CLI flags, output paths, and
  ownership of check/write/inspect behavior;
- every generated artifact has one owner and one reproducibility command;
- no generated artifact is bound to two incompatible command surfaces;
- every runtime, service-adapter, or dispatch path that must change is listed
  in code bindings;
- every test that encodes old behavior is either preserved or explicitly
  migrated to the new behavior;
- every exact code path needed for the change is bound before the audit;
- no design decision is deferred to the implementation plan when it belongs in
  the spec;
- if the spec touches generated code, compile-surface evidence commands are
  defined before the audit;
- if the spec touches sensitive data, privacy and authorization proof commands
  are defined before the audit.

The spec must include a short "pre-audit closure checklist" under the approval
checklist or a task-specific equivalent section. If this checklist is missing
or false, the spec must not proceed to peer audit.

## Required Code Bindings

The spec must bind exact paths for:

- implementation files,
- generated source files,
- codegen input files,
- test files,
- benchmark files,
- review artifacts,
- temporary data directories,
- security-boundary evidence artifacts.

If a path is intentionally deferred, the spec must say why and name the
blocker.

## Generated-Code Requirements

A generated-code spec must define:

- source contract and owner;
- generated crate or module destination;
- deterministic formatting;
- generated API surface;
- unsupported input combinations;
- schema or contract hash behavior when relevant;
- compile-surface budget;
- codegen-check command;
- privacy constraints for generated outputs.

## Storage and Route Requirements

A storage, client route, Agent Service route, or Key Service route spec must
define:

- data classes stored or transmitted;
- plaintext, tokenized, redacted, encrypted, and derived-data boundaries;
- tenant, matter, user, file, chunk, conversation, and audit keys when relevant;
- allowed caller and callee trust zones;
- authorization and audit behavior;
- retry, cancellation, offline, and degraded behavior when relevant;
- allowed copy points;
- forbidden copy points;
- corrupt input behavior;
- wrong-tenant and wrong-matter behavior;
- stale-token and missing-token behavior;
- unauthorized rendering behavior;
- benchmark baseline when performance is claimed.

## Approval Checklist

A spec is not implementation-ready until all are true:

- required reads complete,
- pre-audit closure gate satisfied before the first peer audit,
- measured object precise,
- correctness and privacy oracle defined,
- benchmark method defined when performance is claimed,
- compile-surface budget defined when generated code or dependency-heavy code
  is touched,
- code bindings exact,
- generated and evidence artifact bindings exact,
- test and review bindings exact,
- peer audit passed,
- implementation plan still required before code.

## Stop Gates

Stop if:

- pre-audit closure gate is incomplete,
- the spec depends on unverified library or service behavior,
- sensitive-data, tokenization, or redaction semantics are unclear,
- trust-zone route boundaries are unclear,
- Agent Service behavior is unclear when AI-facing work is in scope,
- Key Service behavior is unclear when rendering or token dictionaries are in
  scope,
- authorization or audit behavior is unclear,
- correctness and privacy oracle is missing,
- benchmark does not isolate the intended measurement,
- compile-surface impact is ignored,
- code bindings are missing,
- peer audit has not passed.
