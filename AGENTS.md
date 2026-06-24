```
WUTHIER TERMINAL PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 WUTHIER TERMINAL. All Rights Reserved.

```

# Wuthier Terminal Agent Contract

Status: active
Scope: `/home/tia/_DEV/RUST/CRATES/wuthier-terminal`

This repository owns the Wuthier Terminal surface: the trusted local client
boundary for a zero-trust legal knowledge platform. Wuthier Terminal mediates
between authorized humans, the AI-facing Agent Service, and the isolated Key
Service. Its contract is to ensure that AI-facing routes operate on tokenized or
redacted content while authorized humans can render sensitive values only
through the approved key and authorization boundary.

This repository is not an experiment dump. It may contain benchmark, review, and
security-analysis artifacts, but every artifact must be reproducible,
evidence-bound, and tied to an explicit contract.

Stack choices are not approved by this file. Database engines, cryptographic
algorithms, model providers, OCR engines, embedding engines, UI frameworks, and
service runtimes remain candidate choices until approved by specs.

## Inderogable Rules

```
1. INTELLECTUAL HONESTY AND MATHEMATICAL RIGOR ARE MANDATORY
2. ALWAYS PROVE THE ASSUMPTION
3. NEVER LIE OR TWEAK RESPONSE
4. NEVER WRITE SOMETHING THAT IS NOT PROVED
5. NEVER USE ICONS OR EMOJIS
6. CLARITY OVER VERBOSITY
7. NEVER USE HYPERBOLIC WORDS OR MARKETING LANGUAGE
8. ALWAYS USE GROUNDED AND HUMBLE TONE
9. MATHILDE MEASURES, NOT PREDICTS
10. DETERMINISM IS MANDATORY WHEN DATA CONTRACTS OR BENCHMARKS DEPEND ON IT
11. PERFORMANCE IS TIME-BOUNDED AND EVIDENCE-BOUND
12. ZERO DUPLICATION IS MANDATORY UNLESS THE SPEC JUSTIFIES A PROTOTYPE COPY
13. NEVER RELAX TEST TOLERANCES BEFORE PROVING THE IMPLEMENTATION IS CORRECT
14. NO LATEX FORMULA FOR MARKDOWN FILES; USE MARKDOWN ENCODING
15. BUILD FOR PERFORMANCE, CORRECTNESS, REPRODUCIBILITY, AND LOW COMPILE SURFACE
16. NO CODE CHANGE BEFORE APPROVED SPEC AND APPROVED IMPLEMENTATION PLAN
17. GENERATED CODE MUST COME ONLY FROM APPROVED CODEGEN
18. SPEED CLAIMS REQUIRE RUN EVIDENCE
19. COMPILE-TIME CLAIMS REQUIRE BUILD EVIDENCE
20. WUTHIER TERMINAL RUNTIME MUST REMAIN SMALL; SERVICE ADAPTERS, BENCHES, TEST FIXTURES, AND DEV TOOLS ARE SEPARATE SURFACES
21. FIRST SPEC DRAFTS MUST CLOSE PRIOR-SPEC, COMMAND, ARTIFACT, DISPATCH, AND TEST-MIGRATION CONTRACTS BEFORE PEER AUDIT
22. AI-FACING ROUTES MUST NEVER RECEIVE PLAINTEXT SENSITIVE VALUES
23. AGENT SERVICE ROUTES MUST NEVER CALL KEY SERVICE ROUTES
24. KEY DICTIONARIES, DECRYPTION KEYS, AND PLAINTEXT RENDERING MUST NEVER BE EXPOSED TO LLM, RAG, EMBEDDING, OR AGENT TOOL SURFACES
25. DOCUMENT AND PROMPT FLOWS MUST TOKENIZE OR REDACT BEFORE AI-FACING STORAGE OR MODEL ACCESS
26. HUMAN RENDERING MUST REQUIRE AUTHORIZATION AND AUDITABLE KEY-SERVICE BOUNDARIES
27. CONVERSATION MEMORY, RETRIEVAL DATA, EMBEDDINGS, AND AGENT LOGS MUST STORE TOKENIZED OR REDACTED CONTENT ONLY
```

**Violation of any rule requires protocol restart.**

## Evidence Discipline

Every non-trivial statement must be grounded by evidence type:

- Code-read evidence: identify the file and observed behavior.
- Run evidence: identify the command and observed output.
- Build evidence: identify command, profile, dirty state, timing, and output.
- Storage evidence: identify the local or remote store, schema, query, and
  observed result.
- Security-boundary evidence: identify the route, caller, callee, trust zone,
  authorization check, and observed allowed or denied behavior.
- Benchmark evidence: identify command, dataset, profile, machine, and result.
- Data-contract evidence: identify source format, schema or classifier contract,
  tokenization contract, generated file if any, and check command.
- External-doc evidence: identify upstream source and date-sensitive claim.
- Hypothesis: state that it is not proved yet and why it is suspected.

No statement may sound certain if it has not been proved.

## Repository Boundaries

Wuthier Terminal owns:

- local trusted client and terminal behavior;
- file, folder, and repository watch contracts when approved;
- matter and client selection contracts when approved;
- prompt tokenization before AI-facing routes;
- response rendering orchestration through the approved Key Service boundary;
- local authorization context handling when approved;
- local audit-event emission when approved;
- redacted or tokenized conversation persistence when approved;
- route isolation between Human Zone, AI Zone, and Key Zone;
- evidence discipline for privacy, security, correctness, and performance.

Wuthier Terminal does not own unless a later approved spec adds that surface:

- final legal advice or legal correctness of model output;
- the Agent Service runtime;
- the Key Service runtime;
- key-management internals;
- model-provider behavior;
- OCR, embedding, database, or cryptographic implementations;
- organization-wide identity policy;
- document-source finality, retention, or legal hold policy;
- stack selection before research, spec, and approval.

Application schemas, service contracts, and deployment manifests may live
outside this repository. External surfaces are allowed only when they pass the
approved Wuthier data-flow, trust-boundary, and evidence contracts.

## Mandatory Execution Flow

Every non-trivial task follows this sequence. No skips.

### 1. Intake (NO CODE)

- Restate the goal precisely.
- Classify the task:
  - research only,
  - spec authoring,
  - peer audit,
  - implementation planning,
  - code implementation,
  - testing or benchmarking,
  - review or documentation.
- List required reads, inputs, datasets, configs, trust boundaries, and
  unknowns.
- Ask the minimum clarifying questions only if the task cannot be specified.

### 2. Context Read (NO CODE)

- Read `AGENTS.md`.
- Read `docs/invariants/core_invariants.md`.
- Read relevant protocols under `docs/protocols/`.
- Read existing specs under `docs/specs/` and reviews under `docs/reviews/`.
- Read target code paths before proposing edits.
- For Wuthier data surfaces, read the relevant upstream contract before using
  the data in tests, security claims, or benchmarks.

### 3. Research Brief (NO CODE)

Before a spec can be written, produce a research brief that states:

- measured object,
- candidate approach,
- Wuthier binding surface,
- trust zones touched,
- sensitive-data classes touched,
- unknowns,
- risks,
- evidence already available,
- evidence required before coding.

### 4. Spec (NO CODE)

The spec is the source of truth.

If a design decision is not in the spec, it must not appear in code.

Every spec must include:

- goal and non-goals,
- source data contract,
- sensitive-data and tokenization contract,
- redaction contract,
- trust-zone and route-isolation contract,
- storage contract,
- authorization and audit contract,
- Agent Service boundary contract when relevant,
- Key Service boundary contract when relevant,
- human rendering contract when relevant,
- generated-artifact or codegen contract when relevant,
- crate and dependency boundaries,
- determinism policy,
- failure contract,
- compile-surface budget,
- performance budget,
- benchmark methodology,
- correctness and privacy proof plan,
- exact code bindings,
- exact test and artifact bindings,
- implementation plan requirement,
- approval status.

Before the first peer audit, every spec draft must complete the pre-audit
closure gate from `docs/protocols/spec_protocol.md`. A peer audit must not be
used to discover basic protocol non-compliance, missing command surfaces,
generated-artifact ownership, dispatch path bindings, prior-spec conflicts, or
test migration from old behavior to new behavior.

### 5. Peer Audit (NO CODE)

Every spec must be audited in a separate review artifact before implementation
planning. The audit must try to falsify the spec and classify exactly:

- `PEER_AUDIT_PASSED`
- `BLOCKED`

### 6. Implementation Plan and Approval (NO CODE)

Before touching code, write an implementation plan that binds:

- files to edit,
- files to create,
- generated files,
- dependency changes,
- security-boundary artifacts,
- benchmark artifacts,
- validation commands,
- expected outputs,
- rollback boundary,
- known risks.

The plan must be approved before any code line is changed.

### 7. Implementation (CODE)

- Implement only the approved spec and plan.
- Keep changes minimal and bounded.
- Preserve deterministic behavior and explicit failure surfaces.
- Do not introduce hidden allocations on hot paths without spec justification.
- Do not use `unwrap`, `expect`, or `panic!` in runtime, codegen, measurement,
  security-boundary, or reusable support logic.
- Do not add environment variables unless the spec defines default, bounds, and
  operator meaning.

### 8. Validation

- Run the narrowest correctness checks first.
- Run privacy and trust-boundary checks before security claims.
- Run deterministic replay checks before performance claims.
- Run build-surface checks before claiming compile-time improvement.
- Run benchmark commands exactly as specified.
- If validation fails, stop and diagnose. Do not tweak expectations.
- Record command, environment, input size, build profile, trust boundary, and
  output.

### 9. Review and Report

- Update review artifacts when work produces evidence.
- Summarize what changed.
- State which invariants are proved.
- State which claims remain unproved.
- Do not claim readiness beyond the completed evidence chain.

## Stop Gates

Work must stop before code if any gate fails:

- Required reads incomplete.
- Source data contract is missing.
- Sensitive-data classification contract is missing when redaction or
  tokenization is in scope.
- Tokenization contract is missing when AI-facing routes are in scope.
- Redaction contract is missing when document or prompt content reaches the
  Agent Service, RAG, embeddings, logs, or LLMs.
- Trust-zone route contract is missing.
- Agent Service boundary contract is missing when AI-facing work is in scope.
- Key Service boundary contract is missing when tokens, keys, dictionaries,
  decryption, or rendering are in scope.
- Storage contract is missing when persistence is in scope.
- Authorization or audit contract is missing when human rendering is in scope.
- Codegen contract is missing when generated artifacts are touched.
- Crate boundary or dependency policy is missing.
- Spec is missing, incomplete, or unapproved.
- Peer audit is missing or blocked.
- Implementation plan is missing or unapproved.
- Exact code/test/artifact bindings are missing.
- Failure contract is missing.
- Benchmark methodology is missing for performance claims.
- Correctness and privacy proof plan is missing.
- Compile-surface budget is missing for generated-code work.
- Dependency choice is justified by preference rather than evidence.

## Mandatory Protocol Reads

Before any non-trivial task, read:

- `docs/protocols/lifecycle_protocol.md`
- `docs/invariants/core_invariants.md`

Then read task-specific protocols:

- research:
  - `docs/protocols/research_protocol.md`
- spec authoring:
  - `docs/protocols/spec_protocol.md`
  - `docs/protocols/peer_audit_protocol.md`
- peer audit:
  - `docs/protocols/peer_audit_protocol.md`
- implementation planning or coding:
  - `docs/protocols/implementation_protocol.md`
  - `docs/protocols/code_style_protocol.md`
  - `docs/protocols/codegen_protocol.md` when generated code is touched
- testing or benchmarking:
  - `docs/protocols/testing_benchmark_protocol.md`
- reviews and reports:
  - `docs/protocols/review_documentation_protocol.md`
- reusable prompts:
  - `docs/protocols/task_prompts.md`

If these reads are incomplete, work must stop.

## Zero-Trust and Trusted-Boundary Rules

Zero-trust, privacy-preserving, secure, isolated, or trusted-client claims must
prove all of:

- plaintext sensitive values are identified before AI-facing transfer;
- tokenization and redaction behavior is deterministic where equality,
  retrieval, audit, or replay depends on it;
- Agent Service routes do not receive plaintext sensitive values;
- RAG, embeddings, conversation memory, retrieval logs, and agent logs do not
  store plaintext sensitive values;
- Key Service routes are unreachable from Agent Service and LLM tool surfaces;
- key dictionaries and decryption keys are isolated from AI-facing routes;
- human rendering requires authorization and emits auditable evidence;
- token collision, token reuse, token rotation, and missing-token behavior are
  explicit when tokens are in scope;
- corrupt, partial, stale, wrong-tenant, wrong-matter, and unauthorized
  behavior is explicit;
- read and write paths do not copy sensitive plaintext beyond spec-allowed
  boundaries;
- test and benchmark inputs separate synthetic data from real sensitive data;
- benchmark comparisons use identical logical payloads;
- warm-cache and cold-cache results are separated when storage is involved.

## Configuration Hygiene

- Do not add configuration knobs unless strictly necessary.
- Every knob must have a spec binding, default, bounds, and operator meaning.
- Avoid environment variables unless the spec explains why a CLI flag or
  checked config object is insufficient.

## Dependency Hygiene

New dependencies require a spec binding that states:

- why the dependency is needed;
- which crate or service owns it;
- which alternatives were considered;
- what version was selected;
- what correctness, privacy, security, or performance risk remains;
- how the dependency is validated in tests.

## Documentation Style

- Use concise markdown.
- Use markdown encoding for formulas and pseudo-formulas.
- Avoid promotional language.
- Keep public claims separate from internal hypotheses.
- Use review documents as due-diligence memos, not conversation traces.
