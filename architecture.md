# wuthier-terminal architecture

This document explains the intended internal architecture of `wuthier-terminal`
as derived from `initial-intake.md`. It is not implementation evidence. Privacy,
security, runtime, compile-time, and performance claims remain bound to specs,
result reviews, and benchmark artifacts.

## Scope

`wuthier-terminal` is the trusted local client boundary for a zero-trust legal
knowledge platform.

It owns, when approved by spec:

- local file, folder, and repository watching;
- local matter and client selection;
- local prompt tokenization before AI-facing calls;
- response rendering orchestration through Key Service;
- local conversation interface behavior;
- local audit-event emission;
- route isolation between Human Zone, AI Zone, and Key Zone.

It does not own by default:

- final legal advice;
- model-provider behavior;
- Agent Service internals;
- Key Service internals;
- database engine choices;
- cryptographic algorithm choices;
- OCR, embedding, or retrieval engine choices;
- organization-wide identity policy.

## Trust Zones

```text
+----------------------------- TRUST ZONES -----------------------------+
|                                                                       |
|  Human Zone                                                           |
|      authorized user                                                  |
|      Wuthier Terminal local client                                    |
|      plaintext may be visible only under approved authorization        |
|                                                                       |
|  AI Zone                                                              |
|      Agent Service                                                    |
|      extraction, OCR, redacted RAG, embeddings, agent tools, LLMs      |
|      plaintext sensitive values are forbidden                         |
|                                                                       |
|  Key Zone                                                             |
|      Key Service                                                      |
|      token dictionaries, decryption authority, rendering authorization |
|      not reachable from AI Zone                                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

The core architectural rule is route separation. Wuthier Terminal may call
Agent Service and Key Service through approved routes. Agent Service must not
call Key Service.

## High-Level Flow

```text
+---------------------------- SYSTEM FLOW ------------------------------+
|                                                                       |
|  plaintext document or prompt                                         |
|       |                                                               |
|       v                                                               |
|  Wuthier Terminal                                                     |
|       |                                                               |
|       +--> sensitive-data detection                                   |
|       +--> tokenization or redaction                                  |
|       |                                                               |
|       v                                                               |
|  Agent Service                                                        |
|       |                                                               |
|       +--> extraction / OCR / chunking when approved                  |
|       +--> redacted retrieval and embeddings                          |
|       +--> LLM orchestration over tokenized/redacted content          |
|       |                                                               |
|       v                                                               |
|  tokenized response                                                   |
|       |                                                               |
|       v                                                               |
|  Wuthier Terminal                                                     |
|       |                                                               |
|       +--> rendering request to Key Service                           |
|       +--> authorization and audit boundary                           |
|       |                                                               |
|       v                                                               |
|  authorized human-visible answer                                      |
|                                                                       |
+-----------------------------------------------------------------------+
```

The AI-facing path sees symbols and redacted text. The human-visible path may
render plaintext only through approved authorization.

## Client Application

The client application is trusted. Its responsibilities are spec-bound and may
include:

- file watching;
- repository watching;
- folder synchronization;
- conversation interface;
- prompt tokenization;
- response rendering orchestration;
- authentication context;
- matter selection;
- client selection;
- local audit-event creation.

The client is the only approved bridge between Agent Service and Key Service
calls. That bridge must not become a service-to-service route.

## Agent Service Boundary

Agent Service is AI-facing. Its responsibilities are candidates from the intake
until specs approve exact behavior:

- file ingestion;
- text extraction;
- OCR;
- chunking;
- redaction verification;
- embedding generation;
- redacted RAG;
- search;
- context retrieval;
- agent tools;
- LLM orchestration.

Agent Service may store only redacted or tokenized documents, chunks,
embeddings, metadata, and audit metadata. It must not store plaintext sensitive
values, decryption keys, or key dictionaries.

## Key Service Boundary

Key Service is isolated. Its responsibilities are candidates from the intake
until specs approve exact behavior:

- key storage;
- token dictionary;
- encryption;
- decryption;
- rendering;
- authorization;
- audit logging.

Key Service must not participate in retrieval, embeddings, or LLM tooling. It
only renders final human-visible responses for authorized requests.

## File Processing Pipeline

```text
+--------------------------- FILE PIPELINE -----------------------------+
|                                                                       |
|  new or changed file                                                  |
|       -> watcher event                                                |
|       -> content identity or version calculation                      |
|       -> extraction path approved by spec                             |
|       -> sensitive-data detection                                     |
|       -> tokenization or redaction                                    |
|       -> redacted document version                                    |
|       -> chunks and embeddings from redacted/tokenized content        |
|       -> raw transfer copy destroyed or retained only by approved rule |
|                                                                       |
+-----------------------------------------------------------------------+
```

Move and delete behavior must be specified before implementation. The intake
suggests that moves can update path metadata without re-indexing, but that
remains a spec decision.

## Prompt and Response Pipeline

```text
+------------------------- PROMPT / RESPONSE ---------------------------+
|                                                                       |
|  user prompt with plaintext                                           |
|       -> local sensitive-data detection                               |
|       -> tokenized prompt                                             |
|       -> LLM request                                                  |
|       -> tokenized model response                                     |
|       -> Key Service render request                                   |
|       -> authorization check                                          |
|       -> audit event                                                  |
|       -> human-visible rendered response                              |
|                                                                       |
+-----------------------------------------------------------------------+
```

Stored conversation memory must remain tokenized or redacted. Rendering is a
view operation for an authorized human, not a reason to store plaintext in
conversation history.

## Tokenization Architecture

Token behavior must be approved by spec before implementation.

The intake establishes the intended behavior:

- every sensitive entity receives a stable token inside an approved scope;
- the same entity maps to the same token inside that scope;
- the LLM receives the token, not the plaintext value;
- token rendering requires Key Service authorization.

Open spec decisions include:

- token namespace;
- tenant, client, matter, or repository scope;
- stability duration;
- collision handling;
- rotation behavior;
- stale-token behavior;
- token dictionary storage;
- audit event shape.

## Storage Architecture

The intake names a candidate RAG database and candidate table families. Those
choices are not approved architecture yet.

Any approved storage design must separate:

- redacted documents;
- redacted chunks;
- embeddings derived from redacted or tokenized content;
- metadata;
- token references;
- conversations;
- retrieval logs;
- audit logs;
- encrypted sensitive values;
- token dictionaries.

The storage contract must prove that AI-facing stores do not contain plaintext
sensitive values.

## Encryption and Key Architecture

The intake lists candidate cryptographic algorithms. This document does not
approve any algorithm.

Any approved encryption design must specify:

- data-key and master-key ownership;
- key wrapping behavior;
- token matching behavior;
- nonce or randomness requirements;
- deterministic or probabilistic behavior where relevant;
- rotation behavior;
- failure behavior;
- audit evidence;
- dependency selection and validation.

## Failure Surfaces

Important failure surfaces must be explicit in specs:

- undetected sensitive value;
- false-positive sensitive value;
- token collision;
- missing token dictionary entry;
- stale token;
- wrong tenant, client, or matter;
- unauthorized rendering;
- Agent Service plaintext rejection;
- Key Service unreachable;
- Agent Service unreachable;
- file watcher missed event;
- partial ingestion;
- corrupt extracted text;
- failed OCR or extraction;
- embedding generation failure;
- retrieval over stale chunks;
- audit write failure;
- response too large;
- cancellation and retry ambiguity.

## Benchmark Boundary

```text
+------------------------- BENCHMARK BOUNDARY -------------------------+
|                                                                       |
|  benchmarks                                                           |
|       -> synthetic datasets                                           |
|       -> file watch workloads                                         |
|       -> detection/tokenization workloads                             |
|       -> redaction and retrieval workloads                            |
|       -> rendering and audit workloads                                |
|       -> evidence JSON and markdown summaries                         |
|                                                                       |
|  production runtime                                                   |
|       -> approved local-client behavior only                          |
|                                                                       |
+-----------------------------------------------------------------------+
```

Architecture diagrams explain intended data movement. They are not performance
claims. Performance claims require benchmark result files with command,
dataset, profile, and output evidence.

## Non-Goals

- No final legal-advice guarantee.
- No predictive legal outcome semantics.
- No undocumented stack lock-in.
- No Agent Service to Key Service route.
- No plaintext sensitive values in AI-facing routes.
- No token dictionary or decryption-key storage in AI-facing surfaces.
- No benchmark fixtures in production runtime.
- No real sensitive data committed to the repository.
