# wuthier-terminal

`wuthier-terminal` is the trusted local client boundary for a zero-trust legal
knowledge platform.

The product intent is defined in `initial-intake.md`: authorized humans work
with real clients, matters, and documents; AI-facing systems work with
tokenized or redacted representations. The Agent Service must not receive
plaintext sensitive values, and the Key Service owns rendering authorization
and token dictionary boundaries.

This repository currently contains the initial Rust crate scaffold plus the
MATHILDE lifecycle, evidence, invariant, protocol, and architecture documents
adapted for Wuthier Terminal.

Stack choices are intentionally not locked here. Database engines,
cryptographic algorithms, model providers, OCR engines, embedding engines, UI
frameworks, and service runtimes require research, specs, peer audit, approved
implementation plans, and evidence before implementation.

## Required Reading

- `AGENTS.md`
- `initial-intake.md`
- `docs/invariants/core_invariants.md`
- `docs/protocols/lifecycle_protocol.md`
- task-specific protocol files under `docs/protocols/`
- `architecture.md`
- `docs/architecture/repository_structure.md`

## Current State

The Rust code is only a minimal executable stub. Product behavior is not
implemented yet.

Do not treat architectural intent as a proved privacy, security, correctness,
or performance claim. Claims require the evidence chain defined by the
protocols.
