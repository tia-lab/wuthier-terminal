```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Lifecycle

Version: 1.0
Status: active
Scope: entire repository

## Purpose

Define the mandatory lifecycle for Wuthier Terminal research, design,
implementation, validation, and review.

Wuthier Terminal is intended to become the trusted local client boundary for a
zero-trust legal knowledge platform. No prototype status permits unclear trust
boundaries, unbound dependencies, unverifiable privacy claims, undocumented
tokenization behavior, manual generated-code edits, or plaintext leakage into
AI-facing surfaces.

## Required Reads

Before any non-trivial task:

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- this protocol
- task-specific protocol under `docs/protocols/`
- existing specs under `docs/specs/`
- existing reviews under `docs/reviews/`
- target code paths when implementation is requested
- upstream service, data, or dependency contracts when behavior matters

## Lifecycle

### Phase 0: Intake

Output:

- goal restatement,
- task class,
- required reads,
- required inputs,
- affected trust zones,
- affected sensitive-data classes,
- unknowns,
- stop/go risks.

No code is allowed.

### Phase 1: Context Read

Output:

- protocols read,
- docs read,
- source files read,
- service or dependency contracts read when behavior matters,
- observed constraints,
- hypotheses still open.

No code is allowed.

### Phase 2: Research Brief

Output path:

- `docs/reviews/[slug]/[slug]_research_brief.md`

No code is allowed.

### Phase 3: Spec

Output path:

- `docs/specs/[slug]_SPEC.md`

The spec must pass `spec_protocol.md`.

No code is allowed.

### Phase 4: Peer Audit

Output path:

- `docs/reviews/[slug]/[slug]_peer_audit.md`

The audit must classify exactly:

- `PEER_AUDIT_PASSED`
- `BLOCKED`

No code is allowed.

### Phase 5: Implementation Plan and Approval

Output path:

- `docs/reviews/[slug]/[slug]_implementation_plan.md`

The plan binds every code file, generated file, dependency change, test,
benchmark, security-boundary artifact, evidence path, and validation command.

No code is allowed until the plan is approved.

### Phase 6: Implementation

Code changes may start only after Phase 5 approval.

Implementation must stay within the approved spec and plan.

### Phase 7: Testing and Benchmarking

Testing follows `testing_benchmark_protocol.md`.

Correctness, privacy-boundary, and determinism evidence are required before
performance claims.

### Phase 8: Result Review

Output path:

- `docs/reviews/[slug]/[slug]_result_review.md`

The result review states:

- what was built,
- what was measured,
- what passed,
- what failed,
- what is proved,
- what remains unproved,
- whether the work should continue, be redesigned, or be promoted.

## Stop Gates

Stop if:

- required reads are incomplete,
- the measured object is unclear,
- source data contract is unclear,
- sensitive-data classes are unclear when detection, redaction, or tokenization
  is in scope,
- tokenization contract is unclear when AI-facing routes are in scope,
- redaction contract is unclear when document, prompt, retrieval, embedding, or
  model routes are in scope,
- trust-zone boundary is unclear,
- Agent Service boundary is unclear when AI-facing work is in scope,
- Key Service boundary is unclear when tokens, dictionaries, decryption, or
  rendering are in scope,
- storage contract is unclear when persistence is in scope,
- authorization or audit contract is unclear when human rendering is in scope,
- generated-code ownership is unclear,
- spec is missing or unapproved,
- peer audit is missing or blocked,
- implementation plan is missing or unapproved,
- benchmark method cannot prove the intended claim,
- compile-surface impact is relevant but unmeasured.
