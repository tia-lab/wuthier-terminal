```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# PROTOCOL: Wuthier Terminal Research

Version: 1.0
Status: active
Scope: research-only work

## Purpose

Research builds enough evidence to decide whether a design deserves a spec. It
does not authorize implementation.

## Required Reads

- `AGENTS.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- this protocol
- relevant existing specs and reviews
- relevant source code
- upstream service, data, or dependency docs when behavior matters

## Research Output

Write:

- `docs/reviews/[slug]/[slug]_research_brief.md`

The brief must include:

- status,
- source materials,
- measured object,
- candidate approach,
- affected trust zones,
- affected sensitive-data classes,
- evidence table,
- hypotheses,
- unknowns,
- risks,
- required decisions before spec,
- recommended next phase.

## Evidence Table

Every brief must separate:

- code-read evidence,
- run evidence,
- build evidence,
- storage evidence,
- security-boundary evidence,
- benchmark evidence,
- data-contract evidence,
- external-doc evidence,
- hypotheses.

## Wuthier Research Minimum

For Wuthier work, research must identify:

- source data and owner;
- sensitive-data classes in scope;
- detection, tokenization, and redaction candidate approach;
- token format, stability, tenant or matter scope, and collision assumptions;
- Human Zone, AI Zone, and Key Zone surfaces touched;
- Agent Service boundary when AI-facing behavior is in scope;
- Key Service boundary when tokens, dictionaries, decryption, or rendering are
  in scope;
- storage boundary for documents, chunks, embeddings, logs, conversations,
  token references, and audit events when relevant;
- authorization and audit requirements when human rendering is in scope;
- generated-code surface when relevant;
- crate, service, and dependency impact;
- compile-surface risk;
- runtime benchmark baseline when performance is claimed;
- correctness and privacy oracle;
- expected failure cases.

## Stop Gates

Stop research escalation if:

- source data ownership is unknown,
- sensitive-data classes cannot be stated,
- tokenization or redaction contract cannot be stated when relevant,
- Agent Service or Key Service boundaries cannot be stated when relevant,
- storage boundary cannot be stated when persistence is in scope,
- authorization or audit behavior cannot be stated when rendering is in scope,
- correctness and privacy oracle is missing,
- compile-surface risk is ignored,
- dependency behavior is assumed but not read,
- performance baseline cannot be defined when performance is claimed.
