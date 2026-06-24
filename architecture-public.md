# Wuthier Terminal Public Architecture

## Purpose

Wuthier Terminal is a preliminary architecture and design for a zero-trust knowledge system that anonymizes, tokenizes, encrypts, and controls sensitive or confidential data for organizations operating under Swiss law, Swiss jurisdiction, or Swiss operational control.

The system is not limited to legal work. Legal, public-sector, healthcare, finance, insurance, fiduciary, education, research, and industrial organizations can have the same core problem: they need AI-assisted search or chat over sensitive data, but they cannot give AI-facing systems uncontrolled plaintext access.

The system is designed so that documents and conversations can be indexed, searched, and used in model context without giving the model direct access to confidential values.

Wuthier Terminal is not an LLM provider. It does not create, train, own, or host foundation models. It controls what data is sent to supported model integrations and how returned tokenized responses are rendered for authorized users.

Zero trust in this document means that an AI-facing component is not trusted with plaintext sensitive values, token dictionaries, decryption keys, or rendering authority. Every route that crosses from the human or key boundary into the AI boundary must be transformed first.

The central rule is:

```text
Humans may work with plaintext when authorized.
AI-facing systems work with tokenized or redacted data.
Key material and token dictionaries are isolated from AI-facing systems.
```

This document describes the intended public architecture. It is not an implementation proof, security certification, or final cryptographic selection.

## Project Principals

Ivo Wuthier (CEO):

- legal and institutional principal for the Wuthier side of the project;
- partner at Wuthier & Nicora / WN Legal according to public firm material;
- attorney, notary, and mediator with work in legal advice and representation before judicial authorities, with focus areas including private and administrative law, and 30 years of experience;
- public links: https://www.wnlegal.ch/about, https://www.linkedin.com/in/ivo-wuthier-67b041b1/.

Mattia Chiesa Wuthier (CTO):

- architect, founder, and lead engineer of Wuthier Terminal;
- degree in mathematics and computer science;
- 20+ years of engineering experience;
- founder and architect of MATHILDE, a deterministic market-measurement data platform;
- public links: https://github.com/tia-lab, https://www.mathilde.dev/, https://github.com/mathilde-trade/.

## Navigation Index

Core framing:

- [Purpose](#purpose)
- [Project Principals](#project-principals)
- [Jurisdictional and Industry Scope](#jurisdictional-and-industry-scope)
- [Definitions](#definitions)
- [Non-Claims](#non-claims)
- [System Components](#system-components)
- [Trust Boundaries](#trust-boundaries)
- [Component Responsibilities](#component-responsibilities)

Data and ingestion:

- [Data Classes](#data-classes)
- [Multi-Signal Sensitive-Data Gate](#multi-signal-sensitive-data-gate)
- [Document Ingestion Flow](#document-ingestion-flow)
- [File Sync and Encryption Diagram](#file-sync-and-encryption-diagram)
- [File Update, Delete, and Move Flow](#file-update-delete-and-move-flow)

AI interaction:

- [Interaction Modes](#interaction-modes)
- [Normal Chat Mode](#normal-chat-mode)
- [Retrieval Mode](#retrieval-mode)
- [Shared Inbound Controls](#shared-inbound-controls)
- [Retrieval Data Flow](#retrieval-data-flow)
- [Model Provider Boundary](#model-provider-boundary)
- [Conversation Memory](#conversation-memory)

Token and encryption:

- [Token Model](#token-model)
- [Token Matching](#token-matching)
- [Encryption and Key System](#encryption-and-key-system)
- [Dual-Secret Rendering Model](#dual-secret-rendering-model)
- [Post-Quantum Direction](#post-quantum-direction)
- [Why Not Homomorphic Encryption for the Main Path](#why-not-homomorphic-encryption-for-the-main-path)
- [Storage Separation](#storage-separation)

Operations and strategy:

- [Network Isolation](#network-isolation)
- [Authorization and Audit](#authorization-and-audit)
- [Threat Model](#threat-model)
- [Failure Cases](#failure-cases)
- [Operational Strategy](#operational-strategy)
- [Market and Partnership Strategy](#market-and-partnership-strategy)
- [Delivery Planning Assumptions](#delivery-planning-assumptions)
- [Research Position](#research-position)
- [External Research Basis](#external-research-basis)
- [Open Technical Contracts](#open-technical-contracts)
- [Summary](#summary)

## Jurisdictional and Industry Scope

The primary jurisdictional assumption is Switzerland.

Architecture consequences:

- remote infrastructure for production use is expected to run on Swiss server infrastructure;
- data storage and data processing for sensitive Swiss workloads should remain under Swiss operational and legal control unless a later deployment contract states otherwise;
- key material and token dictionaries must remain under a Swiss-controlled key boundary;
- the system must be usable by industries that process sensitive or confidential information, not only law firms;
- cross-border processing is treated as a risk requiring explicit legal, technical, and contractual review.

Target sectors:

- public administration;
- cantonal and municipal authorities;
- courts and justice-adjacent organizations;
- law firms and legal departments;
- healthcare and medical administration;
- banks, insurers, and fiduciaries;
- education and research institutions;
- regulated enterprises with confidential technical, commercial, or personal data.

This architecture does not claim legal compliance by itself. It defines a technical separation model intended to support Swiss-law and Swiss-jurisdiction requirements.

## Definitions

Plaintext sensitive value: A real client name, person name, organization name, contract title, address, identifier, secret, credential, or other value that must not be exposed to an AI-facing system.

Token: A symbolic value that replaces a plaintext sensitive value outside the authorized plaintext boundary.

Token dictionary: The mapping between a token and the protected sensitive value. This mapping is owned by the Key Service boundary.

Redacted document: A document representation where sensitive spans have been replaced with tokens or non-sensitive placeholders.

AI-facing system: Any system that performs retrieval, embedding generation, model prompting, agent-tool execution, or model orchestration.

Detection review model: A locally hosted model used only to review sensitive-data detection before content is admitted to AI-facing retrieval or model-provider routes. It is part of the controlled preprocessing boundary, not a general model provider integration.

Rendering: The operation that converts a tokenized answer into a human-readable answer for an authorized user.

Normal chat mode: A conversation mode where the model receives a tokenized prompt but no retrieved document context.

Retrieval mode: A conversation mode where the model receives a tokenized prompt plus redacted or tokenized context retrieved from the AI-facing index.

## Non-Claims

This document does not claim:

- that implementation is complete;
- that all sensitive data can be detected perfectly;
- that AI output is legal advice;
- that Wuthier Terminal creates, trains, owns, or hosts LLM models;
- that any cryptographic algorithm listed here has been selected for production;
- that a database, model provider, UI framework, or service runtime has been selected;
- that performance, privacy, or security properties are proved without test and review evidence.

## System Components

The system has three primary components.

```text
+-------------------+          +-------------------+
|                   |          |                   |
|  Agent Service    |          |    Key Service    |
|                   |          |                   |
|  AI-facing        |          |  key/rendering    |
|  no plaintext     |          |  plaintext gate   |
|                   |          |                   |
+---------^---------+          +---------^---------+
          |                              |
          | tokenized/redacted data      | render request
          |                              |
          v                              v
        +----------------------------------+
        |                                  |
        |     Wuthier Terminal Client      |
        |                                  |
        |     trusted human boundary       |
        |                                  |
        +----------------^-----------------+
                         |
                         | plaintext when authorized
                         v
                    +---------+
                    |  Human  |
                    +---------+
```

Only the Wuthier Terminal Client may coordinate both Agent Service calls and Key Service rendering calls. The Agent Service has no route to the Key Service.

## Trust Boundaries

```text
+----------------------+-----------------------------------------------+
| Boundary             | Allowed data                                  |
+----------------------+-----------------------------------------------+
| Human Boundary       | Plaintext for authorized users                |
| Client Boundary      | Plaintext only where needed for local         |
|                      | detection, tokenization, rendering, and UI    |
| AI Boundary          | Tokenized or redacted data only               |
| Key Boundary         | Token dictionary, encrypted values, key       |
|                      | material, rendering authorization             |
+----------------------+-----------------------------------------------+
```

Forbidden routes:

- Agent Service to Key Service.
- LLM tool to Key Service.
- RAG store to Key Service.
- Embedding pipeline to Key Service.
- Agent Service to token dictionary.
- Model provider to plaintext sensitive values.

Allowed routes:

- Client to Agent Service with tokenized or redacted data.
- Client to Key Service for authorized rendering.
- Human to Client for local interaction.

The zero-trust control is bidirectional:

- outbound to AI: plaintext prompt or document input is detected, tokenized, and redacted before AI-facing access;
- inbound from AI: model output is treated as untrusted tokenized text and can become plaintext only through authorized Key Service rendering;
- storage after either direction remains tokenized or redacted unless a separate plaintext store is explicitly defined with its own isolation controls.

## Component Responsibilities

### Wuthier Terminal Client

The client is the trusted local boundary.

Responsibilities:

- file, folder, and repository observation;
- active client and matter selection;
- local decoding and extraction from source files into a normalized text representation;
- local OCR when needed and when assigned to the client boundary;
- prompt tokenization;
- response rendering orchestration;
- presentation of rendered output to the authorized user;
- audit-event creation for sensitive operations;
- separation of Agent Service and Key Service credentials.

The client must not create a service-to-service bridge where Agent Service can reach Key Service indirectly.

### Agent Service

The Agent Service is AI-facing.

Allowed responsibilities:

- receiving normalized text or Markdown intermediates from the client for controlled preprocessing;
- running the multi-signal sensitive-data gate on the normalized intermediate;
- producing redacted or tokenized document representations;
- chunking;
- embedding generation from redacted or tokenized content;
- retrieval over redacted or tokenized content;
- model context assembly;
- agent-tool orchestration over redacted or tokenized content;
- returning tokenized model responses.

Forbidden responsibilities:

- storing plaintext sensitive values;
- persisting plaintext normalized intermediates after preprocessing;
- persisting original source files;
- storing token dictionaries;
- storing decryption keys;
- calling Key Service;
- rendering tokens to plaintext;
- exposing decryption or rendering as an LLM tool.

### Key Service

The Key Service is the rendering and key boundary.

Responsibilities:

- token dictionary storage;
- encrypted sensitive-value storage;
- data-key wrapping or unwrapping according to the approved key design;
- rendering authorization;
- token rendering;
- audit logging for key and rendering operations.

Forbidden responsibilities:

- retrieval;
- embedding generation;
- model prompting;
- agent-tool execution;
- direct exposure to LLMs;
- accepting calls from Agent Service.

## Data Classes

The system is expected to handle at least these sensitive classes:

- person names;
- client names;
- organization names;
- contract names;
- matter identifiers;
- email addresses;
- phone numbers;
- physical addresses;
- bank identifiers;
- passport numbers;
- government or internal IDs;
- API keys;
- secrets;
- credentials;
- confidential clauses;
- medical or personal context when present.

The exact class list, detection rules, and review thresholds are technical contracts. They are not inferred from this public document.

## Multi-Signal Sensitive-Data Gate

Detection quality is a critical control surface. If a sensitive value is missed before AI-facing transfer, plaintext can cross into the AI boundary. The architecture therefore uses a multi-signal sensitive-data gate before content is admitted to retrieval, embeddings, model-provider prompts, or AI-facing storage.

```text
+---------------------- MULTI-SIGNAL SENSITIVE-DATA GATE ----------------------+
|                                                                               |
|  Input                                                                        |
|    normalized plaintext document or prompt                                    |
|                                                                               |
|  Signal 1: deterministic Rust detectors                                       |
|    exact patterns, configured rules, identifiers, secrets                     |
|                                                                               |
|  Signal 2: tenant/client/matter dictionaries                                  |
|    known parties, aliases, matter names, emails, addresses, terms             |
|                                                                               |
|  Signal 3: document-structure detectors                                       |
|    headers, signatures, party blocks, tables, captions, email metadata        |
|                                                                               |
|  Signal 4: alias, fuzzy, and multilingual matching                            |
|    abbreviations, initials, variants, German/French/Italian/English forms     |
|                                                                               |
|  Signal 5: OCR and extraction confidence                                      |
|    low confidence fails closed or requires review                             |
|                                                                               |
|  Signal 6: controlled local review model                                      |
|    locally hosted in Swiss remote infrastructure                              |
|    no external model-provider call                                            |
|    no training on reviewed content                                            |
|    no persistent plaintext review store                                       |
|                                                                               |
|  Signal 7: cross-document consistency checks                                  |
|    reuse prior matter tokens and detect new variants of known entities        |
|                                                                               |
|  Decision                                                                     |
|    high confidence: emit tokenized/redacted representation                    |
|    uncertain: fail closed or require human review                             |
|    fail: block AI-facing transfer                                             |
|                                                                               |
+-------------------------------------------------------------------------------+
```

### Signal 1: Deterministic Rust Detectors

The deterministic detectors are fast and reproducible.

They are responsible for high-precision detection of classes such as:

- email addresses;
- phone numbers;
- physical addresses;
- IBANs and bank identifiers;
- credit card patterns where relevant;
- passport numbers;
- government or internal IDs;
- API keys;
- secrets;
- credentials;
- configured legal or business terms.

Given the same input, rules, dictionaries, and configuration, deterministic detectors should produce the same tokenization candidates.

### Signal 2: Context Dictionaries

Tenant, client, matter, and workspace dictionaries provide known context.

They may include:

- clients;
- counterparties;
- employees;
- public authorities;
- judges or officers where relevant;
- contract names;
- project names;
- matter names;
- addresses;
- email domains and aliases;
- known internal identifiers.

Known context should have high priority because it catches values that generic named-entity recognition may miss.

### Signal 3: Document-Structure Detection

Sensitive values often appear in predictable document zones.

Examples:

- letterheads;
- signature blocks;
- party definitions;
- invoice tables;
- court captions;
- email headers;
- metadata;
- annex titles;
- spreadsheet column names;
- scanned form fields.

The detector should consider location and structure, not only token text.

### Signal 4: Alias, Fuzzy, and Multilingual Matching

The gate should detect variants of already known entities.

Examples:

- initials;
- abbreviated names;
- spelling variants;
- OCR variants;
- aliases;
- email-derived names;
- German, Swiss-German where applicable, French, Italian, and English variants.

Fuzzy matching must be bounded because aggressive fuzzy matching can create false positives. Uncertain matches should go to risk scoring or human review.

### Signal 5: OCR and Extraction Confidence

Detection cannot be stronger than the extracted text.

If OCR or extraction confidence is low, the document must not move directly to AI-facing routes. Low confidence should fail closed or require human review according to policy.

### Signal 6: Controlled Local Review Model

The controlled local review model is a review step for misses that deterministic and structural signals may not catch.

It may review:

- legal entities;
- relationships between parties;
- confidential paragraphs;
- medical or personal context;
- context-sensitive identifiers;
- unusual references that look harmless in isolation but are sensitive in the document context.

This review model is local to the controlled Swiss remote infrastructure. It is not a supported external model integration. It must not train on reviewed content, call external model APIs, write persistent plaintext review logs, or expose plaintext to the Agent Service retrieval path.

The review model's output is not trusted as final truth. It is a second signal used to update the redaction/tokenization candidate set. If the review is uncertain, the route must fail closed or require human review according to policy.

### Signal 7: Cross-Document Consistency

Within a tenant, client, or matter scope, prior tokenization decisions should inform later detection.

If a value was tokenized earlier, later documents should detect variants of the same value where possible.

Examples:

- `John Doe`;
- `J. Doe`;
- `Mr Doe`;
- related email aliases;
- related company names;
- repeated contract labels.

Cross-document consistency reduces missed variants and keeps token use stable across a matter.

### Risk Scoring and Human Review

Signals should produce a risk decision, not only a binary match list.

Possible outcomes:

- high-confidence tokenization;
- high-confidence non-sensitive text;
- uncertain span requiring human review;
- low extraction confidence requiring review;
- blocked transfer.

For high-risk sectors or public-sector data, policy may require human review for uncertain spans before any AI-facing transfer.

### Admission Rule

Content can enter AI-facing retrieval, embeddings, normal chat, or model-provider routes only after the multi-signal gate emits a tokenized or redacted representation.

```text
plaintext input
  -> deterministic detectors
  -> context dictionaries
  -> structure detectors
  -> alias/fuzzy/multilingual matching
  -> OCR confidence gate
  -> controlled local review model
  -> cross-document consistency checks
  -> risk scoring
  -> fail closed or human review if uncertain
  -> tokenization/redaction decision
  -> AI-facing representation
```

The architecture does not claim perfect detection. No detection system should claim a 100 percent guarantee for all languages, formats, and document contexts. The control objective is to reduce false negatives to the lowest measurable level through deterministic detection, known-context dictionaries, document-structure detection, alias and fuzzy matching, OCR confidence gates, controlled local review, cross-document consistency checks, fail-closed behavior for uncertainty, human review where policy requires it, and language- and domain-specific validation.

## Document Ingestion Flow

```text
1. Local filesystem provides source files
2. Source files may include PDF, DOCX, CSV, email, TXT, Markdown, scanned files, or other supported formats
3. Client observes the file, folder, or repository event
4. Client assigns tenant/client/matter/file context
5. Client decodes or extracts the source file locally
6. Client produces a normalized plaintext intermediate
7. The normalized intermediate is TXT or Markdown; the final choice is a technical contract
8. Client ships the normalized intermediate to the controlled Swiss remote preprocessing boundary
9. Remote preprocessing runs the multi-signal sensitive-data gate
10. Remote preprocessing produces tokenization/redaction decisions
11. Token dictionary entries are created or reused through the key boundary
12. Sensitive values are encrypted through the multi-level envelope model
13. Sensitive spans are replaced by tokens or redactions
14. Redacted/tokenized document representation is produced
15. Redacted chunks are produced
16. Embeddings are produced from redacted or tokenized chunks
17. AI-facing index stores redacted chunks, embeddings, and metadata
18. Key boundary stores token mappings, encrypted sensitive values, and wrapped keys
19. Remote plaintext normalized intermediate is destroyed after preprocessing
```

Data movement:

```text
Local source file
  |
  v
Client local decode / extract
  |
  v
Normalized plaintext intermediate
  |
  v
Swiss remote preprocessing boundary
  |
  +--> Key Service path: token dictionary, encrypted sensitive values, wrapped keys
  |
  +--> Agent Service path: redacted document, chunks, embeddings, metadata
  |
  +--> destroy remote plaintext intermediate
```

The remote plaintext intermediate is temporary processing material. It must not remain persisted on the server after tokenization, redaction, encryption, and derived outputs are complete. The Agent Service receives the representation needed for retrieval and model context. It does not receive the token dictionary or plaintext values.

### File Sync and Encryption Diagram

```text
+-------------------+       +----------------------+       +----------------------+
|                   |       |                      |       |                      |
| Source files      | ----> | Wuthier Terminal     | ----> | Normalized plaintext |
| PDF / DOCX / CSV  |       | client watcher       |       | intermediate         |
| mail / TXT / MD   |       | + local extraction   |       | TXT or Markdown      |
|                   |       |                      |       |                      |
+-------------------+       +----------+-----------+       +----------+-----------+
                                      |                              |
                                      v                              v
                           +----------------------+       +----------------------+
                           |                      |       |                      |
                           | File version record  | ----> | Swiss remote         |
                           | content identity     |       | preprocessing        |
                           | matter/client scope  |       | boundary             |
                           |                      |       |                      |
                           +----------------------+       +----------+-----------+
                                                                  |
                                                                  v
                           +----------------------+       +----------------------+
                           |                      |       |                      |
                           | Deterministic Rust   | ----> | Controlled local     |
                           | detector             |       | review model         |
                           |                      |       | Swiss infrastructure |
                           +----------+-----------+       +----------+-----------+
                                      |                              |
                                      +--------------+---------------+
                                                     |
                                                     v
                                      +------------------------------+
                                      |                              |
                                      | Tokenization / redaction     |
                                      |                              |
                                      +--------------+---------------+
                                                     |
                         +---------------------------+---------------------------+
                         |                                                       |
                         v                                                       v
          +------------------------------+                        +------------------------------+
          |                              |                        |                              |
          | Agent Service path           |                        | Key Service path              |
          |                              |                        |                              |
          | redacted document            |                        | token dictionary              |
          | redacted chunks              |                        | encrypted sensitive values    |
          | embeddings from redacted     |                        | wrapped data keys             |
          | content                      |                        | access policy                 |
          | retrieval metadata           |                        | audit records                 |
          |                              |                        |                              |
          +------------------------------+                        +------------------------------+
                         |                                                       ^
                         |                                                       |
                         v                                                       |
          +------------------------------+                        +------------------------------+
          |                              |                        |                              |
          | AI-facing retrieval index    |                        | Multi-level envelope          |
          | no plaintext                 |                        | encryption                    |
          | no token dictionary          |                        |                              |
          | no keys                      |                        | sensitive value -> Data Key   |
          |                              |                        | Data Key -> Master Key wrap   |
          +------------------------------+                        | Master Key -> Key Service     |
                                                                  | protection                   |
                                                                  |                              |
                                                                  +------------------------------+

Temporary remote plaintext:
  normalized intermediate is destroyed after encrypted/tokenized outputs are created
```

The sync output is split. The Agent Service path receives only redacted or tokenized data. The Key Service path receives token dictionary records and encrypted sensitive values protected by the multi-level envelope model. The remote TXT/Markdown intermediate is temporary and is destroyed after preprocessing.

## File Update, Delete, and Move Flow

File update:

```text
changed file
  -> content identity check
  -> new file version
  -> local decode / extract to normalized plaintext intermediate
  -> remote multi-signal sensitive-data gate and tokenization
  -> redacted chunks regenerated
  -> embeddings regenerated
  -> encrypted sensitive records created or updated
  -> remote plaintext intermediate destroyed
  -> prior version retained or removed according to retention policy
```

File delete:

```text
deleted file
  -> retrieval entry disabled or removed
  -> embeddings disabled or removed
  -> token mappings retained or deleted according to retention policy
  -> audit event recorded
```

File move:

```text
moved file
  -> path metadata update
  -> no content reprocessing if content identity is unchanged
  -> audit event recorded
```

Retention behavior is not a public assumption. It must be defined by policy and technical contract.

## Interaction Modes

Wuthier Terminal supports two distinct AI interaction modes. They share the same zero-trust boundary rules.

### Normal Chat Mode

Normal chat uses the model without document retrieval.

```text
1. Human writes prompt
2. Client detects sensitive values in prompt
3. Multi-signal sensitive-data gate runs when policy requires it or when deterministic detection is uncertain
4. Client replaces sensitive values with existing or new tokens
5. Client sends tokenized prompt to Agent Service
6. Agent Service sends tokenized prompt to selected model integration
7. Model returns tokenized response
8. Client treats model response as untrusted text
9. Client requests rendering for allowed tokens from Key Service
10. Key Service checks authorization
11. Key Service resolves allowed tokens
12. Client presents rendered answer to human
13. Tokenized conversation memory and audit events are written
```

Normal chat route:

```text
Human plaintext prompt
  -> Client tokenization
  -> Agent Service tokenized prompt
  -> supported model integration
  -> tokenized model response
  -> Client render request
  -> Key Service authorization
  -> Human rendered response
```

### Retrieval Mode

Retrieval mode adds redacted or tokenized document context.

```text
1. Human writes prompt
2. Client detects sensitive values in prompt
3. Multi-signal sensitive-data gate runs when policy requires it or when deterministic detection is uncertain
4. Client replaces sensitive values with existing or new tokens
5. Client sends tokenized prompt to Agent Service
6. Agent Service searches redacted/tokenized index
7. Agent Service builds context from redacted/tokenized chunks
8. Agent Service sends tokenized prompt and redacted context to selected model integration
9. Model returns tokenized response
10. Client treats model response as untrusted text
11. Client requests rendering for allowed tokens from Key Service
12. Key Service checks authorization
13. Key Service resolves allowed tokens
14. Client presents rendered answer to human
15. Tokenized conversation memory, retrieval logs, and audit events are written
```

Retrieval route:

```text
Human plaintext prompt
  -> Client tokenization
  -> Agent Service tokenized prompt
  -> redacted/tokenized retrieval
  -> supported model integration
  -> tokenized model response
  -> Client render request
  -> Key Service authorization
  -> Human rendered response
```

### Shared Inbound Controls

Both modes use the same inbound controls.

```text
model response
  -> treated as untrusted text
  -> token references extracted
  -> requested token set checked against user/client/matter scope
  -> Key Service authorization
  -> allowed tokens rendered
  -> rendered view shown to human
  -> tokenized response retained for memory
```

The model cannot authorize rendering. The model cannot create a trusted token dictionary entry. The model cannot request arbitrary token rendering outside the client, user, client, matter, and policy scope.

## Retrieval Data Flow

The retrieval system stores and searches redacted or tokenized content.

Example:

```text
Stored redacted chunk:
The agreement was signed by {{person:8f31}} on {{date:aa21}}.
```

The model can reason over:

- chronology;
- document structure;
- references between entities;
- obligations;
- conflicts;
- prior similar matters;
- legal arguments.

The model does not need the plaintext values for those operations.

## Model Provider Boundary

Wuthier Terminal does not own the underlying LLM models.

Model execution is provided by supported model integrations. A user or organization may choose among supported models where policy, availability, and deployment configuration allow it.

The model-provider boundary is AI-facing. It may receive only tokenized prompts and redacted or tokenized context. It must not receive token dictionaries, decryption keys, or plaintext sensitive values.

Model-provider selection must not weaken the data boundary. Changing the model changes the model endpoint, behavior, and operational risk, but it does not change the rule that plaintext sensitive values are excluded from AI-facing inputs.

## Conversation Memory

Conversation memory follows the same boundary as retrieval.

Stored:

```text
Analyse the position of {{person:8f31}} in matter {{matter:9b12}}.
```

Not stored:

```text
Analyse the position of John Doe in matter Smith acquisition.
```

Rendered text may be shown to an authorized human, but it must not become the stored conversation record unless a later policy explicitly allows a plaintext store with its own isolation and authorization controls.

## Token Model

Token example:

```text
John Doe -> {{person:8f31}}
```

Token properties that must be defined by technical contract:

- namespace;
- tenant scope;
- client scope;
- matter scope;
- stability period;
- collision resistance;
- rotation behavior;
- revocation behavior;
- missing-token behavior;
- rendering authorization behavior.

The token is not the secret. The secret is the mapping:

```text
{{person:8f31}} -> John Doe
```

That mapping belongs to the Key Service boundary.

## Token Matching

The intake identifies deterministic keyed matching as a required direction.

Candidate construction:

```text
normalized sensitive value
  -> keyed digest
  -> token suffix or lookup handle
```

Candidate primitive:

```text
HMAC-SHA3-512
```

Purpose:

- stable matching of the same sensitive value inside an approved scope;
- no plaintext comparison in the AI-facing system;
- no unauthenticated token dictionary reconstruction.

Open technical decisions:

- normalization rules;
- key ownership;
- tenant and matter scoping;
- truncation length;
- collision handling;
- rotation behavior.

## Encryption and Key System

Sensitive values are stored encrypted inside the key boundary. The AI-facing boundary stores token references, not encrypted sensitive values.

The encryption model is multi-level envelope encryption:

```text
+----------------------------- ENCRYPTION LEVELS -----------------------------+
|                                                                             |
|  Level 1: Sensitive value encryption                                         |
|                                                                             |
|      plaintext sensitive value                                               |
|          -> encrypted with Data Key                                          |
|          -> encrypted sensitive value                                        |
|                                                                             |
|  Level 2: Data-key wrapping                                                  |
|                                                                             |
|      Data Key                                                               |
|          -> wrapped with Master Key material                                 |
|          -> wrapped data key or key handle                                   |
|                                                                             |
|  Level 3: Master-key protection                                              |
|                                                                             |
|      Master Key material                                                     |
|          -> protected by Key Service boundary                                |
|          -> constrained by authorization, audit, and dual-secret controls    |
|                                                                             |
|  Additional cryptographic surfaces                                           |
|                                                                             |
|      token matching: HMAC-SHA3-512 candidate                                 |
|      post-quantum key wrapping: ML-KEM-1024 candidate                        |
|      post-quantum signatures: ML-DSA or SLH-DSA candidate                    |
|                                                                             |
+-----------------------------------------------------------------------------+
```

The three encryption levels are separate from the AI-facing token reference. A token can be stored in the AI-facing system; the encrypted value, wrapped key, and master-key protection stay in the key boundary.

### Encrypted Value Record

Each protected value is represented by a key-boundary record.

```text
token_id
tenant_scope
client_scope
matter_scope
sensitive_class
encrypted_sensitive_value
wrapped_data_key_or_key_handle
algorithm_id
key_version
normalization_version
associated_data
created_at
rotated_at
audit_metadata
```

The exact field names are implementation details. The separation is not: the AI-facing system may reference `token_id`, but it must not store `encrypted_sensitive_value`, `wrapped_data_key_or_key_handle`, key material, or dictionary records.

### Envelope Encryption Flow

```text
1. plaintext sensitive value enters the client/key boundary
2. value is classified by sensitive class
3. value is normalized for matching if deterministic matching is required
4. token lookup handle is produced inside the key boundary
5. token_id is created or reused
6. data key is generated or selected according to key policy
7. associated data is built from tenant, client, matter, token_id, class, and version
8. plaintext sensitive value is encrypted with authenticated encryption
9. data key is wrapped or referenced through the dual-secret key design
10. encrypted record is stored in the Key Service boundary
11. token_id is returned to the client for use in redacted/tokenized content
```

Boxed flow:

```text
+--------------------+       +----------------------+       +-------------------+
|                    |       |                      |       |                   |
| Plaintext value    | ----> | Sensitive classifier | ----> | Token assignment  |
|                    |       | and normalizer       |       |                   |
+--------------------+       +----------------------+       +---------+---------+
                                                                        |
                                                                        v
+--------------------+       +----------------------+       +-------------------+
|                    |       |                      |       |                   |
| Key Service store  | <---- | Multi-level envelope | <---- | Data Key          |
|                    |       | encryption           |       | generation        |
+---------+----------+       +----------+-----------+       +-------------------+
          |                             |
          | encrypted value             | wrapped data key / key handle
          v                             v
  encrypted_sensitive_value     wrapped_data_key_or_key_handle

AI-facing output:
  token_id only
```

Data relation:

```text
plaintext sensitive value
  -> AEAD(data_key, associated_data)
  -> encrypted_sensitive_value

data_key
  -> dual-secret wrapping or key-handle policy
  -> wrapped_data_key_or_key_handle

token_id
  -> reference used outside key boundary
```

Candidate data-encryption primitives:

```text
AES-256-GCM-SIV
XChaCha20-Poly1305
```

The selected primitive must provide authenticated encryption. The final choice requires technical validation, implementation review, and test evidence.

### Associated Data

Authenticated encryption must bind the ciphertext to the context that makes the value valid.

Candidate associated-data fields:

```text
tenant_id
client_id
matter_id
token_id
sensitive_class
algorithm_id
key_version
normalization_version
record_version
```

If associated data does not match at render time, decryption must fail.

### Rendering and Decryption Flow

```text
1. client has tokenized model response
2. client extracts token_ids requested for rendering
3. client sends render request with user, tenant, client, matter, token_ids, and Authority A proof
4. Key Service authenticates caller
5. Key Service checks policy for user, client, matter, token class, and operation
6. Key Service obtains or proves Authority B
7. Key Service verifies that Authority A and Authority B satisfy the dual-secret policy
8. Key Service unwraps or accesses the data key
9. Key Service decrypts encrypted_sensitive_value with associated data
10. Key Service returns only authorized plaintext values or a rendered response fragment
11. Key Service writes audit record
12. client presents rendered output to the human
```

The decrypted value is a rendering result. It is not sent to Agent Service, model providers, retrieval, embeddings, or AI-facing logs.

Boxed render path:

```text
+----------------------+        +----------------------+        +----------------------+
|                      |        |                      |        |                      |
| Tokenized response   | -----> | Client render request| -----> | Key Service policy   |
|                      |        | + Authority A proof  |        | check                |
+----------------------+        +----------------------+        +----------+-----------+
                                                                             |
                                                                             v
+----------------------+        +----------------------+        +----------------------+
|                      |        |                      |        |                      |
| Human rendered view  | <----- | Plaintext result     | <----- | Dual-secret unwrap   |
|                      |        | for allowed tokens   |        | + AEAD decrypt       |
+----------------------+        +----------------------+        +----------------------+

Forbidden outputs:
  Agent Service plaintext
  model-provider plaintext
  retrieval plaintext
  embedding plaintext
  AI-facing log plaintext
```

### Key Rotation

The key system must support versioned records.

Rotation-relevant fields:

```text
algorithm_id
key_version
normalization_version
record_version
rotated_at
previous_key_reference when policy allows it
```

Rotation must not change token meaning without a defined migration path. A token can remain stable while the encrypted value record and wrapped key version change.

## Dual-Secret Rendering Model

Rendering requires two independent authorities.

```text
Secret or authority A:
  client-side, user-side, or device-side authority

Secret or authority B:
  Key Service authority

rendering allowed only if:
  A is valid
  B is valid
  requested tokens are allowed for the user, client, and matter
```

Intended effect:

- Agent Service has neither authority.
- AI-facing database compromise has neither authority.
- Token dictionary access alone is not sufficient to render plaintext.
- Client context alone is not sufficient to render plaintext.
- Key Service authority alone is not sufficient to render plaintext unless the final approved key design explicitly defines a break-glass or recovery route.
- Rendering is scoped by user, tenant, client, matter, token class, and audit policy.

The exact split is a technical contract. Acceptable designs must preserve the property that no AI-facing component can obtain both authorities.

Possible authority splits include:

- user-held or client-held secret plus Key Service wrapping secret;
- client authorization token plus Key Service key material;
- hardware-backed local key plus remote key-wrapping authority.

The public architecture requires split authority. It does not select the final mechanism.

### Dual-Secret Storage Shape

The storage shape must keep the encrypted value and the authorities separate.

```text
Key record:
  token_id
  encrypted_sensitive_value
  wrapped_data_key_or_key_handle
  policy_scope
  audit_metadata

Authority A:
  held or proven by client/user/device boundary

Authority B:
  held or proven by Key Service boundary

Plaintext recovery:
  requires token_id
  requires policy authorization
  requires Authority A
  requires Authority B
  emits audit record
```

The Agent Service may hold `token_id` references. It must not hold Authority A, Authority B, encrypted sensitive values, wrapped data keys, or token dictionary records.

## Post-Quantum Direction

The intake identifies post-quantum key and signature primitives as design candidates.

Candidate key-wrapping direction:

```text
ML-KEM-1024
```

Candidate signature direction:

```text
ML-DSA
SLH-DSA
```

These are not production selections in this document. Their use requires:

- implementation maturity review;
- interoperability review;
- key-size and latency measurement;
- migration and rotation plan;
- failure behavior;
- auditability;
- compatibility with the dual-secret rendering model.

## Why Not Homomorphic Encryption for the Main Path

Homomorphic encryption is relevant to sensitive-data AI, but it is not the main control in this architecture.

Homomorphic encryption allows computation over encrypted data without decrypting the data first. In abstract form:

```text
plaintext
  -> encrypt
  -> ciphertext
  -> compute over ciphertext
  -> encrypted result
  -> decrypt result
```

That model is attractive because an AI provider would not see plaintext inputs. The issue is practical scope. Full LLM and RAG workflows require more than arithmetic over encrypted values:

- document parsing;
- sensitive-span detection;
- token matching;
- embedding generation;
- vector search;
- retrieval context assembly;
- long-context model inference;
- tool routing;
- response rendering and authorization.

Fully homomorphic execution of this whole pipeline is not the selected architecture because it is not currently the most practical way to provide low-latency, auditable, Swiss-controlled document search and chat over mixed legal, administrative, financial, and technical content.

Wuthier Terminal uses a different control:

```text
plaintext
  -> detect
  -> tokenize / redact
  -> AI works on tokenized or redacted representation
  -> Key Service renders only authorized tokens
```

This keeps the AI-facing path useful while removing plaintext sensitive values from prompts, retrieval context, embeddings, and conversation memory.

Homomorphic encryption remains a possible future research area for narrow operations where it is technically and operationally justified. It is not assumed as the core privacy mechanism for this architecture.

## Storage Separation

The system separates AI-facing storage from key storage.

```text
+-----------------------------+-------------------------------+
| AI-facing store             | Key store                     |
+-----------------------------+-------------------------------+
| redacted documents          | token dictionary              |
| redacted chunks             | encrypted sensitive values    |
| embeddings from redacted    | wrapped data keys             |
| content                     | access policies               |
| metadata                    | user permissions              |
| retrieval logs              | rendering audit records       |
| token references            | key access audit records      |
| tokenized conversations     |                               |
+-----------------------------+-------------------------------+
```

Forbidden:

- plaintext sensitive values in AI-facing store;
- decryption keys in AI-facing store;
- token dictionary in AI-facing store;
- embeddings generated from plaintext sensitive values;
- conversation memory containing plaintext sensitive values.

## Network Isolation

Required route shape:

```text
Client -> Agent Service
Client -> Key Service
Agent Service -> Key Service: forbidden
LLM tool -> Key Service: forbidden
RAG store -> Key Service: forbidden
Embedding worker -> Key Service: forbidden
```

Isolation requirements:

- separate service credentials;
- separate storage credentials;
- separate network policy;
- separate audit trails;
- no shared decryption route;
- no shared token dictionary route.

## Authorization and Audit

Rendering requires authorization.

Authorization inputs may include:

- user identity;
- tenant;
- client;
- matter;
- document;
- token class;
- requested operation;
- session state;
- policy constraints.

Audit events should record:

- actor;
- tenant/client/matter context;
- operation;
- token class;
- authorization decision;
- timestamp;
- service boundary used;
- failure reason when denied.

Audit records should not store unnecessary plaintext.

## Threat Model

### LLM Compromise

Expected exposure:

- tokenized prompts;
- redacted retrieved context;
- tokenized responses.

Expected non-exposure:

- token dictionary;
- decryption keys;
- plaintext sensitive values.

### Agent Service Compromise

Expected exposure:

- redacted documents;
- redacted chunks;
- embeddings generated from redacted or tokenized content;
- AI-facing metadata;
- AI-facing audit metadata.

Expected non-exposure:

- token dictionary;
- Key Service credentials;
- decryption keys;
- plaintext sensitive values.

### AI-Facing Database Compromise

Expected exposure:

- tokenized or redacted stored data;
- embeddings derived from tokenized or redacted content;
- metadata allowed in the AI-facing boundary.

Expected non-exposure:

- plaintext sensitive values;
- key dictionary;
- decryption keys.

### Embedding Leakage

Embedding leakage is treated as data exposure. The embedding input must be tokenized or redacted before generation. The design must not rely on embeddings to hide plaintext that was present in the input.

### Key Service Compromise

The Key Service is the critical plaintext recovery boundary.

Expected risk:

- token dictionary exposure;
- encrypted sensitive-value exposure;
- key-material exposure depending on final design;
- rendering policy exposure.

Mitigations must come from:

- dual-secret rendering;
- key wrapping;
- authorization policy;
- audit trail;
- key rotation;
- network isolation;
- operational controls.

### Client Compromise

The client is trusted. If the local client or authorized user session is compromised, plaintext may be exposed.

Mitigations must come from:

- local authentication;
- session controls;
- least-privilege token rendering;
- audit;
- device security;
- optional local secret protection.

## Failure Cases

The architecture must define explicit behavior for:

- undetected sensitive value;
- false-positive detection;
- token collision;
- stale token;
- missing token dictionary entry;
- wrong tenant;
- wrong client;
- wrong matter;
- unauthorized rendering;
- Key Service unavailable;
- Agent Service unavailable;
- partial ingestion;
- failed OCR or extraction;
- embedding failure;
- audit write failure;
- retention-policy conflict;
- deleted file with existing token references;
- moved file with unchanged content;
- user cancellation.

Failure must not silently downgrade to plaintext AI-facing processing.

## Operational Strategy

The intended operating model is:

```text
1. Keep plaintext handling local or inside the key boundary.
2. Keep AI-facing systems useful through stable symbols and redacted context.
3. Keep key rendering narrow, authorized, and auditable.
4. Keep service routes separated.
5. Measure correctness, privacy boundaries, and performance before claims.
```

This is an engineering constraint, not promotional language.

## Market and Partnership Strategy

The first market should be Swiss organizations with sensitive or legally protected data and a clear need for AI-assisted document work.

Priority partnership targets:

- Swiss federal bodies;
- cantonal administrations;
- municipal administrations where sensitive citizen data is processed;
- courts and justice-adjacent public bodies;
- public hospitals or health-administration bodies;
- regulated private organizations that must satisfy Swiss confidentiality and data-protection expectations.

The strategic public-sector objective is to test whether this architecture can become a reference pattern for sensitive-data AI work in Switzerland.

Partnership goals:

- validate the trust-boundary model with real public-sector requirements;
- validate Swiss infrastructure and key-sovereignty requirements;
- validate normal chat and retrieval mode with non-public data;
- define audit evidence acceptable to public-sector stakeholders;
- determine whether a federal or cantonal body can recommend, procure, or standardize the architecture for sensitive AI workflows.

The architecture cannot enforce itself as a default. Default status would require institutional adoption, procurement decisions, security review, legal review, and operational evidence.

## Delivery Planning Assumptions

Initial engineering planning estimates:

```text
MVP engineering target:        2 to 3 months
Initial production target:     approximately 6 months
Public-sector procurement use: requires separate review and evidence
```

MVP means a narrow controlled system:

- trusted local client;
- normal chat mode;
- retrieval mode over a limited corpus;
- deterministic tokenization path for selected data classes;
- Key Service rendering path;
- Swiss-hosted remote services;
- audit events for ingestion, prompt tokenization, retrieval, and rendering;
- synthetic or controlled non-production data.

Initial production means a hardened technical system:

- broader sensitive-data class coverage;
- stronger authorization model;
- key rotation and recovery behavior;
- operational monitoring;
- deployment hardening;
- legal and security review;
- public-sector or regulated-industry pilot evidence;
- documented failure behavior.

These timeframes are planning assumptions, not commitments. They depend on team size, procurement constraints, security review, infrastructure access, and the exact data classes in scope.

The 2 to 3 month MVP estimate does not mean readiness for real citizen data. The approximately 6 month initial production estimate does not by itself mean public-sector procurement readiness. Use with real citizen data requires additional evidence, including key-service review, dual-secret rendering validation, Swiss infrastructure deployment evidence, audit evidence, legal review, security review, operational procedures, and approval by the responsible institution.

## Research Position

The problem is not only model quality. The unresolved operational problem is how to let AI systems use sensitive organizational knowledge while preventing AI-facing infrastructure from receiving plaintext sensitive values or key authority.

Existing approaches often reduce only part of the risk:

- normal SaaS AI chat can expose plaintext prompts;
- normal RAG can store plaintext chunks or embeddings derived from plaintext;
- provider-side encryption does not remove provider access if the provider can decrypt for processing;
- local-only systems reduce remote exposure but may not solve collaboration, retrieval, audit, or operational deployment needs;
- generic redaction can remove too much context or fail to preserve stable entity relationships.

The market hypothesis is that current solutions often fall into three incomplete patterns:

```text
SaaS RAG with data residency
  solves deployment convenience
  may leave plaintext prompts, chunks, embeddings, or provider-side processing exposed

Fully local or air-gapped AI
  reduces external exposure
  can limit usability, collaboration, model choice, operations, and audit integration

Generic pseudonymization or redaction
  reduces direct exposure
  can break entity relationships across documents and matters
```

Wuthier Terminal is designed to address these three constraints together: useful AI interaction, Swiss-controlled infrastructure, and stable entity-preserving tokenization with isolated rendering authority. This is a research and design thesis, not proof of market uniqueness.

Wuthier Terminal's research thesis is that the following combination is necessary for the target class of Swiss sensitive-data workloads:

```text
trusted local client
  + deterministic tokenization
  + redacted retrieval
  + isolated Key Service
  + multi-level envelope encryption
  + dual-secret rendering authority
  + Swiss server infrastructure
  + audit evidence
```

This document does not claim that Wuthier Terminal is the only possible architecture. A uniqueness claim requires comparative technical and market research. The current claim is narrower: this architecture directly addresses the plaintext exposure, key-sovereignty, and AI/RAG retrieval problems as one combined system.

## External Research Basis

The architecture is consistent with these public research and policy observations:

- Swiss data-protection law protects personality and fundamental rights where personal data is processed.
- Swiss Government Cloud planning includes Swiss data storage and data processing for sovereignty requirements, and a federal private-cloud level with data processed in federal data centres.
- Public reporting on Privatim's November 2025 position states that international SaaS use for sensitive personal data or data subject to secrecy by public bodies is only possible when the responsible body encrypts the data and the provider has no key access.
- European data-protection material on LLMs identifies retrieval-augmented generation as a method that lets LLM systems reference specific documents, which is the class of system where plaintext retrieval context becomes a material privacy issue.

Research links:

- https://www.fedlex.admin.ch/eli/cc/2022/491/en
- https://www.bit.admin.ch/en/sgc-en
- https://interoperable-europe.ec.europa.eu/collection/open-source-observatory-osor/news/swiss-privacy-regulator-establishes-new-compliance-standards
- https://www.edpb.europa.eu/system/files/2025-04/ai-privacy-risks-and-mitigations-in-llms.pdf

## Open Technical Contracts

The following contracts must be specified before implementation decisions:

- sensitive-data class taxonomy;
- deterministic detection rules;
- local AI review rules;
- false-negative and false-positive validation by language, including German, Swiss-German where applicable, French, Italian, and English;
- false-negative and false-positive validation by document type, including legal, administrative, financial, medical, and technical documents where in scope;
- token namespace and format;
- token matching key ownership;
- token scope and rotation;
- token dictionary schema;
- dual-secret rendering mechanism;
- data encryption primitive;
- post-quantum wrapping and signature use;
- supported model-provider list;
- model-selection policy;
- Agent Service API;
- Key Service API;
- storage schema;
- audit schema;
- authorization policy;
- retention and deletion policy;
- network isolation policy;
- benchmark and validation methodology.

## Summary

Wuthier Terminal separates three operations:

```text
AI work:
  tokenized or redacted data

Key work:
  token dictionary, encrypted values, authorization, rendering

Human work:
  plaintext visible only when authorized
```

The Agent Service receives redacted or tokenized inputs. The Key Service owns the mapping back to sensitive values. The client coordinates the two boundaries without allowing the Agent Service to reach the Key Service.

The design goal is not to make sensitive data disappear. The goal is to keep plaintext access inside the boundaries where it is required and auditable, while giving AI-facing systems only the representation they need to perform retrieval and reasoning tasks.
