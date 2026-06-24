```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# Wuthier Terminal Core Invariants

Status: active
Scope: entire repository

These invariants are mandatory unless a later approved spec explicitly narrows
or supersedes one. Supersession must be local, justified, and audited.

## Trust-Zone Invariants

1. The Human Zone contains the authorized user and trusted local client.
2. The AI Zone contains Agent Service, RAG, embeddings, agent tools, and LLM
   providers.
3. The Key Zone contains key dictionaries, decryption authority, rendering
   authorization, and key audit trails.
4. AI Zone components must not receive plaintext sensitive values.
5. AI Zone components must not call Key Zone routes.
6. Key Zone functionality must not be exposed as an LLM tool.
7. Human-visible plaintext rendering must pass through an approved
   authorization boundary.
8. Trust-zone boundaries must be documented before implementation.
9. Trust-zone violations are correctness failures, not operational warnings.

## Sensitive-Data Invariants

1. Sensitive data classes must be declared by spec before detection or
   redaction behavior is implemented.
2. Plaintext sensitive values may exist only inside approved Human Zone or Key
   Zone boundaries.
3. Document content must be tokenized or redacted before AI-facing storage,
   embeddings, retrieval, model access, or agent-tool access.
4. User prompts must be tokenized before AI-facing model or tool access.
5. AI responses must remain tokenized until an authorized rendering boundary.
6. Conversation memory must store tokenized or redacted content only.
7. Retrieval logs, agent logs, embeddings, and telemetry must not store
   plaintext sensitive values.
8. Synthetic data must be clearly marked and must not be confused with real
   sensitive data.
9. Real sensitive data must not be committed to the repository.

## Tokenization and Redaction Invariants

1. Token format, namespace, stability, and scope must be defined by spec.
2. Token equality semantics must be deterministic when retrieval, audit, or
   replay depends on equality.
3. Token dictionaries must be isolated from AI-facing routes.
4. Token collision behavior must be explicit.
5. Missing-token behavior must be explicit.
6. Token rotation, revocation, and stale-token behavior must be explicit when
   supported.
7. Redaction must preserve enough non-sensitive context for approved retrieval
   use cases and must not invent facts.
8. Local AI review for sensitive-data detection is optional until approved by
   spec and must not leak plaintext to AI Zone services.
9. Tokenization and redaction tests must cover false-positive and false-negative
   behavior when the feature is in scope.

## Storage Invariants

1. AI-facing stores must contain only redacted or tokenized document content.
2. Embedding stores must be built from redacted or tokenized content only.
3. Key dictionaries must be stored separately from AI-facing stores.
4. Encryption keys and plaintext render values must not be stored in AI-facing
   stores.
5. Storage schemas must identify tenant, matter, file, version, chunk,
   conversation, retrieval, and audit boundaries when those entities are in
   scope.
6. Database engine choice is not approved by these invariants.
7. Storage durability, retention, deletion, legal hold, and audit semantics must
   be spec-bound before implementation.
8. Stale, corrupt, partial, wrong-tenant, wrong-matter, and unauthorized records
   must have explicit failure behavior.

## Client and Route Invariants

1. Wuthier Terminal is the trusted local client surface unless a later approved
   spec splits that responsibility.
2. File, folder, and repository watching must have explicit event semantics
   before implementation.
3. File create, update, delete, and move flows must be deterministic where
   indexing or audit depends on ordering.
4. Matter and client selection must be explicit before retrieval or rendering.
5. Agent Service routes receive redacted or tokenized inputs only.
6. Key Service routes receive token-rendering requests only from approved
   callers.
7. Agent Service credentials and Key Service credentials must be separate.
8. No route may silently downgrade authorization, redaction, or tokenization
   failure.
9. Offline, degraded, retry, and cancellation behavior must be spec-bound when
   the route is in scope.

## Code Boundary Invariants

1. Runtime code must stay small and focused on approved local-client behavior.
2. Agent adapters, Key Service adapters, OCR adapters, embedding adapters,
   database adapters, benchmarks, and fixtures are separate surfaces unless a
   spec proves otherwise.
3. Dependency-heavy behavior must not enter the default runtime surface without
   a spec.
4. Generated files are not edited by hand.
5. Generated code must come only from approved codegen.
6. Public APIs must be intentional and spec-bound.
7. Configuration knobs must be spec-bound with defaults, bounds, and operator
   meaning.
8. Security-sensitive code paths must use explicit error handling.

## Testing and Benchmark Invariants

1. Privacy and trust-boundary claims require run evidence.
2. Performance claims require run evidence.
3. Compile-time claims require build evidence.
4. Correctness, privacy, and determinism must pass before speed claims.
5. Benchmark datasets must identify whether they are synthetic, anonymized, or
   real approved data.
6. Real sensitive data must not be used in tests or benchmarks unless a spec
   defines storage, access, cleanup, and audit controls.
7. Baselines must use identical logical payloads.
8. Warm-cache and cold-cache results must be separated when storage is
   involved.
9. Failed and unstable runs are evidence and must not be hidden.
10. Tolerances must not be relaxed before proving correctness.

## Documentation Invariants

1. Specs are the source of truth.
2. Peer audits are separate artifacts.
3. Implementation plans are separate artifacts.
4. Result reviews record evidence and limitations.
5. Documentation must separate proved facts from hypotheses.
6. Conversation trace is not documentation.
7. Public claims must not exceed recorded evidence.
8. A first spec draft must already close prior-spec conflicts, command
   surfaces, generated-artifact ownership, dispatch path bindings, exact code
   bindings, and test migration from old behavior to new behavior.
9. A peer audit must not be used as a substitute for author-side spec
   completeness. If basic protocol compliance is missing, the work restarts at
   spec authoring.
