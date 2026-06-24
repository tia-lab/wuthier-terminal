```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# Wuthier Terminal Protocol Prompts

These prompts are reusable task starters. They do not override `AGENTS.md` or
the protocols.

## Research Prompt

```text
Read AGENTS.md, docs/invariants/core_invariants.md,
docs/protocols/lifecycle_protocol.md, and
docs/protocols/research_protocol.md first.

Task: prepare a research brief for [slug].

Do not write code. Do not change dependencies. Produce
docs/reviews/[slug]/[slug]_research_brief.md with measured object, source
materials, affected trust zones, sensitive-data classes, evidence table,
hypotheses, unknowns, risks, required decisions before spec, and recommended
next phase.
```

## Spec Prompt

```text
Read AGENTS.md, core_invariants.md, lifecycle_protocol.md,
research_protocol.md, spec_protocol.md, peer_audit_protocol.md, the research
brief, and relevant code.

Task: write docs/specs/[slug]_SPEC.md.

Do not write code. The spec must bind source data, sensitive-data classes,
tokenization, redaction, trust zones, Agent Service boundaries, Key Service
boundaries, storage, prompt and conversation behavior, rendering authorization
and audit, generated artifacts when relevant, crate and service boundaries,
dependencies, determinism, failure behavior, correctness and privacy oracle,
benchmarks, compile-surface budget, runtime budget, code paths, artifacts,
tests, review paths, and approval checklist.
```

## Peer Audit Prompt

```text
Read AGENTS.md, core_invariants.md, lifecycle_protocol.md,
spec_protocol.md, peer_audit_protocol.md, the research brief, and the target
spec.

Task: audit docs/specs/[slug]_SPEC.md.

Do not edit the spec in the first pass. Try to falsify measured object clarity,
source data ownership, sensitive-data coverage, tokenization semantics,
redaction semantics, trust-zone boundaries, Agent Service plaintext exclusion,
Key Service isolation, storage plaintext exclusion, authorization and audit,
crate isolation, compile-surface budget, dependency containment, correctness
and privacy oracle, benchmark isolation, failure behavior, and code binding
completeness. Write docs/reviews/[slug]/[slug]_peer_audit.md and classify
exactly PEER_AUDIT_PASSED or BLOCKED.
```

## Implementation Plan Prompt

```text
Read AGENTS.md, core_invariants.md, lifecycle_protocol.md,
implementation_protocol.md, code_style_protocol.md, testing_benchmark_protocol.md,
the approved spec, and the peer audit.

Task: write docs/reviews/[slug]/[slug]_implementation_plan.md.

Do not write code. Bind every file to edit/create, generated artifact,
dependency change, test, benchmark, security-boundary evidence path, command,
expected output, risk, and validation step. End with an explicit approval gate
before code.
```

## Implementation Prompt

```text
Read AGENTS.md, implementation_protocol.md, code_style_protocol.md,
codegen_protocol.md when relevant, testing_benchmark_protocol.md, the approved
spec, and the approved implementation plan.

Task: implement only the approved plan.

Do not add behavior absent from the spec. Keep changes minimal. After code,
perform the pre-test audit required by implementation_protocol.md and report
whether testing may begin.
```

## Test and Benchmark Prompt

```text
Read AGENTS.md, testing_benchmark_protocol.md, the approved spec, approved
implementation plan, and implemented code paths.

Task: run the bound tests and benchmarks.

Do not change expected values, tolerances, or benchmark scope unless
independent evidence proves the original expectation was wrong. Record command,
machine metadata, source data identity, trust zones, sensitive-data classes,
storage identity when relevant, dataset identity, correctness, privacy,
determinism, compile-surface output when relevant, benchmark output, and
limitations. Prepare the result review.
```
