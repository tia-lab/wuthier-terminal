# wuthier-terminal

# Zero-Trust Legal Knowledge Platform
## Privacy-Preserving AI Architecture for Legal, Compliance and Professional Services

---

# Vision

The objective is to transform a traditional filesystem into an AI-native knowledge layer where artificial intelligence can reason over documents without ever accessing sensitive information.

Humans operate on real names, real clients, real contracts, and real entities.

LLMs operate on symbolic representations of those entities.

The system acts as a translation layer between human reality and AI reality.

At no point should an LLM require access to confidential client information.

---

# Fundamental Principle

Traditional AI systems work like this:

Plaintext Documents
→ RAG
→ LLM
→ Answer

This architecture works like this:

Plaintext Documents
→ Classification
→ Tokenization
→ Encryption
→ Redacted RAG
→ LLM
→ Tokenized Response
→ Human Rendering
→ Final Answer

The AI never sees sensitive information.

The human always sees sensitive information if authorized.

---

# High-Level Architecture

The system consists of three major components.

## Component 1: Client Application

Runs on the user's computer.

Responsibilities:

- File watching
- Repository watching
- Folder synchronization
- Conversation interface
- Prompt tokenization
- Response rendering
- Authentication
- Matter selection
- Client selection

The client application acts as a bridge between:

- Human
- Agent Service
- Key Service

The client application is trusted.

---

## Component 2: Agent Service (Remote)

This is the AI-facing environment.

The Agent Service is responsible for:

- File ingestion
- Text extraction
- OCR
- Chunking
- Redaction
- Embedding generation
- RAG
- Search
- Context retrieval
- Agent tools
- LLM orchestration

The Agent Service stores:

- Redacted documents
- Redacted chunks
- Embeddings
- Metadata
- Audit metadata

The Agent Service never stores:

- Plaintext sensitive values
- Decryption keys
- Key dictionaries

The Agent Service cannot decrypt anything.

The Agent Service cannot call the Key Service.

The Agent Service cannot retrieve plaintext.

---

## Component 3: Key Service (Remote)

This is completely isolated.

Responsibilities:

- Key storage
- Token dictionary
- Encryption
- Decryption
- Rendering
- Authorization
- Audit logging

The Key Service stores:

- Encrypted values
- Token mappings
- Access policies
- User permissions
- Audit trails

The Key Service never exposes functionality to any LLM.

The Key Service never participates in retrieval.

The Key Service never participates in embeddings.

The Key Service only renders final human-visible responses.

---

# Trust Boundaries

The system contains three trust zones.

## Human Zone

Trusted.

Contains:

- User
- Local application

Can view plaintext.

---

## AI Zone

Semi-trusted.

Contains:

- Agent Service
- RAG
- Embeddings
- LLM providers

Cannot view plaintext.

Cannot decrypt.

Only sees tokens.

---

## Key Zone

Highly trusted.

Contains:

- Encryption keys
- Dictionary mappings
- Decryption service

Can reveal plaintext.

Cannot be reached by AI.

---

# File Processing Pipeline

## New File

When a file is added:

1. Client watcher detects file.
2. File is synchronized.
3. Agent Service extracts text.
4. Sensitive data is identified.
5. Tokens are generated.
6. Sensitive spans are replaced.
7. Redacted document is created.
8. Chunks are generated.
9. Embeddings are created.
10. Raw uploaded copy is destroyed.

Only redacted representations remain.

---

## File Update

When a file changes:

1. Watcher detects change.
2. Content hash is recalculated.
3. New version is created.
4. Text is re-extracted.
5. Sensitive values are re-tokenized.
6. Chunks are regenerated.
7. Embeddings are regenerated.
8. Previous version is archived if required.

---

## File Delete

When a file is deleted:

1. Watcher detects deletion.
2. File is marked deleted.
3. Chunks are removed from retrieval.
4. Embeddings are removed.
5. Audit trail remains.

---

## File Move

When a file is moved:

1. Watcher detects move.
2. Path metadata updates.
3. No re-index required.
4. No re-embedding required.

---

# Sensitive Information Detection

The system uses two stages.

## Stage 1: Deterministic Detection

Detect:

- Names
- Emails
- Phone numbers
- Addresses
- IBANs
- Credit cards
- Passport numbers
- IDs
- API keys
- Secrets

This stage provides precision.

---

## Stage 2: Local AI Review

A local model may review:

- Legal entities
- Relationships
- Confidential paragraphs
- Medical information
- Context-sensitive data

This stage provides recall.

---

# Tokenization System

Every sensitive entity receives a stable token.

Example:

John Doe

becomes

{{person:8f31}}

The same entity always maps to the same token inside a tenant.

Example:

John Doe
→ {{person:8f31}}

John Doe again
→ {{person:8f31}}

This enables search without revealing identity.

---

# Prompt Tokenization

Example:

User writes:

"Please analyse the legal case against John Doe."

Before reaching the LLM:

John Doe
→ {{person:8f31}}

Prompt becomes:

"Please analyse the legal case against {{person:8f31}}."

Only the tokenized prompt reaches the model.

---

# Retrieval and RAG

The RAG system only stores redacted content.

Example:

"The agreement was signed by {{person:8f31}}."

The model can still reason over:

- Relationships
- Patterns
- Similar cases
- Legal arguments
- Prior documents

without knowing the actual identity.

---

# RAG Database Design

Recommended database:

Turso / libSQL

Reasons:

- Fast
- Embedded vectors
- SQL
- Simple deployment
- Multi-region support
- Good Rust ecosystem

Core tables:

- tenants
- matters
- files
- file_versions
- documents
- chunks
- embeddings
- token_references
- conversations
- retrieval_logs
- audit_logs

The RAG database never contains plaintext sensitive information.

---

# Key Dictionary

Separate database.

Stores:

token_id
→ encrypted value

Example:

{{person:8f31}}
→ encrypted("John Doe")

{{contract:2ad1}}
→ encrypted("Share Purchase Agreement")

Only the Key Service can access this mapping.

---

# Encryption Architecture

The system uses envelope encryption.

Sensitive Value
→ encrypted using Data Key

Data Key
→ wrapped using Master Key

Master Key
→ protected by Key Service

Recommended algorithms:

Data Encryption:
- AES-256-GCM-SIV
or
- XChaCha20-Poly1305

Token Matching:
- HMAC-SHA3-512

Post-Quantum Key Wrapping:
- ML-KEM-1024

Post-Quantum Signatures:
- ML-DSA
or
- SLH-DSA

---

# Response Flow

LLM produces:

"The claim against {{person:8f31}} appears weak."

This response returns to the client.

The client calls the Key Service.

The Key Service checks:

- User identity
- Matter permissions
- Client permissions
- Audit requirements

If authorized:

{{person:8f31}}
→ John Doe

Final rendered answer:

"The claim against John Doe appears weak."

Only the human sees the rendered version.

---

# Conversation Memory

Conversation history is stored only in tokenized form.

Stored:

"Analyse the case against {{person:8f31}}."

Never stored:

"Analyse the case against John Doe."

This prevents future leakage through retrieval.

---

# Security Guarantees

Compromise of the LLM:

- No plaintext access

Compromise of the Agent Service:

- No plaintext access

Compromise of the RAG database:

- No plaintext access

Compromise of embeddings:

- No plaintext access

Only compromise of the Key Service can reveal plaintext.

The Key Service is therefore isolated.

---

# Network Isolation

Agent Service:

- Separate process
- Separate database
- Separate credentials
- Separate network

Key Service:

- Separate process
- Separate database
- Separate credentials
- Separate network

No route exists from Agent Service to Key Service.

No AI component can invoke decryption.

Only the client application can access both systems.

---

# Final Principle

Humans operate on reality.

AI operates on encrypted symbolic reality.

The platform continuously translates between those two worlds while ensuring that confidential information never reaches the model.

This creates a zero-trust legal knowledge platform where AI can reason over sensitive legal data without ever possessing the sensitive data itself.
