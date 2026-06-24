```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Peer Audit

Version: 1.0
Status: active
Scope: spec and result audits

## Purpose

The peer audit tries to falsify the work before code starts or before claims
are accepted.

Default mode is findings-first and no-edit.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- `docs/protocols/spec_protocol.md`
- target research brief
- target spec
- relevant source code
- relevant service or dependency docs when behavior is material

## Spec Audit Lenses

The audit must challenge:

- pre-audit closure gate completeness,
- measured object clarity,
- source data ownership,
- sensitive-data class coverage,
- tokenization semantics,
- redaction semantics,
- Human Zone, AI Zone, and Key Zone boundaries,
- Agent Service reachability and input guarantees,
- Key Service reachability, dictionary isolation, and rendering guarantees,
- authorization and audit behavior,
- storage boundary and plaintext exclusion,
- conversation memory and retrieval-log behavior,
- generated-code determinism when relevant,
- generated-code compile surface when relevant,
- crate and service boundary isolation,
- dependency containment,
- correctness and privacy oracle,
- benchmark isolation,
- performance budget,
- failure behavior,
- code binding completeness,
- generated and evidence artifact binding completeness,
- client or operator interpretation safety.

## Output

Write:

- `docs/reviews/[slug]/[slug]_peer_audit.md`

Classify exactly one:

- `PEER_AUDIT_PASSED`
- `BLOCKED`

## Stop Gates

Block the spec if:

- the pre-audit closure gate is missing or false,
- a core assumption is unproved,
- benchmark cannot prove the intended claim,
- correctness or privacy oracle is weak,
- sensitive-data, tokenization, or redaction behavior is ambiguous,
- trust-zone route boundaries are under-specified,
- Agent Service boundary is under-specified when AI-facing work is in scope,
- Key Service boundary is under-specified when tokens or rendering are in scope,
- authorization or audit behavior is under-specified,
- storage boundary could allow plaintext leakage,
- codegen behavior is under-specified when generated code is touched,
- compile-surface impact is ignored,
- code bindings are incomplete,
- dependency behavior is assumed but not verified.
