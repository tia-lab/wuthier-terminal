```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# Wuthier Terminal Repository Structure

Status: initial architecture lock

## Purpose

This document records the intended structure for the Wuthier Terminal
repository. It is an architecture guardrail, not an implementation plan.

Stack choices remain deferred to approved specs.

## Current Workspace Shape

```text
Cargo.toml
  package: wuthier-terminal
  current root Rust crate

src/main.rs
  current executable stub

bin
  repository maintenance tools only

docs
  architecture, invariants, protocols, specs, reviews, benchmark evidence

initial-intake.md
  product intake and initial system model
```

## Target Workspace Shape

The exact split is not approved. A future spec may keep a single crate or split
the repository into crates and services. If a split is approved, the target
shape should preserve these ownership boundaries:

```text
crates/wuthier-terminal
  trusted local client, terminal entrypoints, local configuration, local
  matter/client selection, prompt tokenization, response rendering orchestration

crates/wuthier-core
  shared value types, error types, trust-zone route contracts, token and
  redaction interfaces, audit event interfaces

crates/wuthier-watch
  file, folder, and repository watch behavior when approved

crates/wuthier-agent-client
  Agent Service client adapter; AI-facing inputs only

crates/wuthier-key-client
  Key Service client adapter; rendering and token dictionary routes only

benchmarks
  standalone benchmark crates and synthetic datasets, outside production
  runtime members when approved

fixtures
  synthetic test data only, never real sensitive data

docs
  architecture, invariants, protocols, specs, reviews, benchmark evidence

bin
  repository maintenance tools only
```

This target shape is a boundary model. It is not approval to create these
crates.

## Ownership Model

Wuthier Terminal is the trusted local client boundary. It does not replace the
Agent Service or Key Service.

```text
Human Zone
  owns authorized human interaction
  may view plaintext through approved local and key-service rendering

Wuthier Terminal repository
  owns local client behavior
  owns prompt tokenization before AI-facing calls
  owns response rendering orchestration through Key Service
  owns file/repository watch behavior when approved
  owns local matter/client selection when approved
  owns local audit event emission when approved
  owns evidence discipline for privacy, correctness, and performance

Agent Service
  owns extraction, OCR, chunking, redacted retrieval, embeddings, search, agent
  tools, and model orchestration when approved outside or inside a later
  service-specific spec
  must not receive plaintext sensitive values
  must not call Key Service

Key Service
  owns token dictionaries, decryption authority, rendering authorization, and
  key audit behavior when approved outside or inside a later service-specific
  spec
  must not be exposed to AI-facing tools
```

## Data-Flow Model

Plaintext may enter Wuthier Terminal only through approved Human Zone inputs.

```text
human plaintext document or prompt
  -> local sensitive-data detection
  -> local tokenization or redaction
  -> AI-facing request with tokenized/redacted content
  -> Agent Service retrieval or model orchestration
  -> tokenized response
  -> Wuthier Terminal rendering request to Key Service
  -> authorized human-visible plaintext response
```

The Agent Service and AI-facing stores operate on tokenized or redacted content
only. The Key Service owns the dictionary and rendering authority. Wuthier
Terminal coordinates both boundaries but must not create an Agent-to-Key route.

## Storage Boundary

Approved specs must separate:

- tokenized or redacted document text;
- chunks and retrieval metadata;
- embeddings derived from tokenized or redacted content;
- token references;
- conversation memory;
- audit events;
- key dictionaries;
- encrypted sensitive values;
- plaintext render buffers.

Database engine choices are intentionally unspecified here. Any database choice
requires a spec binding, dependency contract, storage schema, failure contract,
privacy proof, and validation command.

## Compile-Surface Rule

No crate should compile unrelated surfaces.

Examples:

- Runtime users should not compile OCR, embedding, database, or network
  adapters unless the approved feature needs them.
- Agent Service client code should not compile Key Service internals.
- Key Service client code should not compile model-provider or retrieval
  tooling.
- Benchmarks and fixtures must not be part of the default production runtime
  compile path.
- Synthetic-data generators must not become runtime requirements.

## Schema and Contract Ownership

Schemas and contracts may live outside this repository.

Allowed ownership shapes:

```text
application or client crate
  local data types and client route contracts

service repository
  Agent Service or Key Service API contracts

schema repository
  shared interface definitions when approved

Wuthier Terminal repository
  client-owned contracts, test schemas, and compatibility fixtures only
```

No Rust-only annotation may define security-critical behavior unless a spec
approves that mechanism and its validation path.

## Non-goals

- No undocumented stack lock-in.
- No Agent Service to Key Service route.
- No plaintext sensitive values in AI-facing routes.
- No token dictionary or decryption-key storage in AI-facing surfaces.
- No benchmark support inside production library crates.
- No generated code edited by hand.
- No real sensitive data committed as fixtures.
- No public privacy, security, performance, or compile-time claim without
  recorded evidence.
