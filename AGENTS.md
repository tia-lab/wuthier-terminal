```
MATHILDE PROPRIETARY AND CONFIDENTIAL
Copyright (c) 2024 MATHILDE. All Rights Reserved.

This document contains trade secrets and confidential information owned
exclusively by MATHILDE, protected under Swiss law (URG, UWG, Art. 162 StGB).

PROHIBITED: Reproduction, copying, distribution, disclosure, or derivative
works without prior written authorization from MATHILDE.

ACCESS REQUIREMENT: Executed NDA with MATHILDE required. Unauthorized access
or possession violates Swiss law. Violations subject to civil remedies,
injunctive relief, damages, and criminal prosecution.

Legal Contact: massimo.nicora@wnlegal.ch
```

# MBT Cache Agent Contract

Status: active
Scope: `/home/tia/_DEV/MATHILDE/mbt-cache`

This repository owns the production MBT-native cache database surface. MBT
Cache stores validated MBT payload bytes in Heed and stores only generated
lookup keys and predicate columns in SQLite. The public runtime must expose one
coherent cache API for insert, upsert, delete, latest, range, search, and
time-machine flows.

This repository is not an experiment dump. It may contain benchmark and review
artifacts, but every artifact must be reproducible, evidence-bound, and tied to
an explicit contract.

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
20. MBT CACHE RUNTIME MUST REMAIN SMALL; CODEGEN, BENCHES, TEST SCHEMAS, AND DEV FIXTURES ARE SEPARATE SURFACES
21. FIRST SPEC DRAFTS MUST CLOSE PRIOR-SPEC, COMMAND, ARTIFACT, DISPATCH, AND TEST-MIGRATION CONTRACTS BEFORE PEER AUDIT
22. NO SERDE PAYLOAD PATH IN CACHE RUNTIME
23. NO OLD MONOLITHIC MBT DEPENDENCY
24. SQLITE MUST NEVER STORE MBT PAYLOAD BYTES
25. LATEST AND RANGE MUST USE HEED PAYLOAD ROUTES DIRECTLY
26. SEARCH AND TIME-MACHINE MUST USE SQLITE FOR KEY-ONLY HIT DISCOVERY AND HEED FOR PAYLOAD REPLAY
27. TIME-MACHINE CONTEXT MUST BE BUILT FROM HEED, NOT FROM SQLITE PAYLOADS
```

**Violation of any rule requires protocol restart.**

## Evidence Discipline

Every non-trivial statement must be grounded by evidence type:

- Code-read evidence: identify the file and observed behavior.
- Run evidence: identify the command and observed output.
- Build evidence: identify command, profile, dirty state, timing, and output.
- DB evidence: identify Heed or SQLite surface, schema, query, and observed result.
- Benchmark evidence: identify command, dataset, profile, machine, and result.
- Schema evidence: identify proto file, generated file, schema hash, and check command.
- External-doc evidence: identify upstream source and date-sensitive claim.
- Hypothesis: state that it is not proved yet and why it is suspected.

No statement may sound certain if it has not been proved.

## Repository Boundaries

MBT Cache owns:

- MATHILDE cache option interpretation when cache options are approved;
- descriptor-to-cache schema model contracts;
- generated Heed key contracts;
- generated SQLite lookup table contracts;
- generated predicate-column binders for SQLite;
- generated key-only search and time-machine hit discovery;
- direct Heed latest and range payload routes;
- unified insert, upsert, delete, batch seed, latest, range, search, and
  time-machine runtime semantics;
- benchmark and evidence discipline for cache behavior.

MBT Cache does not own:

- the MBT wire/archive format;
- MBT transport options;
- JSON, protobuf, CSV, Arrow, Parquet, compression, or transponding adapters;
- Aggregator, Primitives, Regime, or `math` source-data finality;
- exchange finality, watermark, closed-bar, or hole-repair semantics;
- application-specific predicate policy beyond generated searchable fields;
- application-specific schema ownership.

Application schemas may live outside this repository. External schema folders
are allowed when they import approved MBT and MBT Cache options and pass codegen
checks.

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
- List required reads, inputs, datasets, configs, and unknowns.
- Ask the minimum clarifying questions only if the task cannot be specified.

### 2. Context Read (NO CODE)

- Read `AGENTS.md`.
- Read `docs/invariants/core_invariants.md`.
- Read relevant protocols under `docs/protocols/`.
- Read existing specs under `docs/specs/` and reviews under `docs/reviews/`.
- Read target code paths before proposing edits.
- For MATHILDE data surfaces, read the relevant upstream contract before using
  the data in tests or benchmarks.

### 3. Research Brief (NO CODE)

Before a spec can be written, produce a research brief that states:

- measured object,
- candidate approach,
- MBT Cache binding surface,
- unknowns,
- risks,
- evidence already available,
- evidence required before coding.

### 4. Spec (NO CODE)

The spec is the source of truth.

If a design decision is not in the spec, it must not appear in code.

Every spec must include:

- goal and non-goals,
- source schema contract,
- cache schema contract,
- Heed payload contract,
- SQLite lookup contract,
- query route contract,
- MBT I/O and trusted-access contract,
- codegen contract,
- crate and dependency boundaries,
- determinism policy,
- failure contract,
- compile-surface budget,
- performance budget,
- benchmark methodology,
- correctness proof plan,
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
  or reusable support logic.
- Do not add environment variables unless the spec defines default, bounds, and
  operator meaning.

### 8. Validation

- Run the narrowest correctness checks first.
- Run deterministic replay checks before performance claims.
- Run build-surface checks before claiming compile-time improvement.
- Run benchmark commands exactly as specified.
- If validation fails, stop and diagnose. Do not tweak expectations.
- Record command, environment, input size, build profile, and output.

### 9. Review and Report

- Update review artifacts when work produces evidence.
- Summarize what changed.
- State which invariants are proved.
- State which claims remain unproved.
- Do not claim readiness beyond the completed evidence chain.

## Stop Gates

Work must stop before code if any gate fails:

- Required reads incomplete.
- Source schema contract is missing.
- Cache schema contract is missing.
- Heed payload contract is missing.
- SQLite lookup contract is missing when search or time-machine is in scope.
- Query route contract is missing when reads are in scope.
- MBT I/O or trusted-access contract is missing when MBT bytes are in scope.
- Codegen contract is missing.
- Crate boundary or dependency policy is missing.
- Spec is missing, incomplete, or unapproved.
- Peer audit is missing or blocked.
- Implementation plan is missing or unapproved.
- Exact code/test/artifact bindings are missing.
- Failure contract is missing.
- Benchmark methodology is missing.
- Correctness proof plan is missing.
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

## Zero-Copy and Trusted-Access Rules

Zero-copy, low-copy, or trusted-access claims must prove all of:

- MBT payload bytes match the declared schema;
- archived data can be validated before trusted access;
- trusted access is unsafe only at explicitly documented boundaries;
- SQLite stores key-only hit data and searchable predicate columns, never MBT
  payload bytes;
- latest and range routes replay payload bytes directly from Heed;
- search and time-machine routes use SQLite only for key discovery;
- time-machine context is fetched from Heed by deterministic key/range logic;
- read and write paths do not copy payloads beyond spec-allowed copy points;
- row and key ordering are deterministic when storage is involved;
- benchmark comparisons use identical logical payloads;
- warm-cache and cold-cache results are separated when storage is involved;
- corrupt, partial, old-version, wrong-schema, and missing-row behavior is explicit.

## Configuration Hygiene

- Do not add configuration knobs unless strictly necessary.
- Every knob must have a spec binding, default, bounds, and operator meaning.
- Avoid environment variables unless the spec explains why a CLI flag or
  checked config object is insufficient.

## Dependency Hygiene

New dependencies require a spec binding that states:

- why the dependency is needed;
- which crate owns it;
- which alternatives were considered;
- what version was selected;
- what correctness or performance risk remains;
- how the dependency is validated in tests.

## Documentation Style

- Use concise markdown.
- Use markdown encoding for formulas and pseudo-formulas.
- Avoid promotional language.
- Keep public claims separate from internal hypotheses.
- Use review documents as due-diligence memos, not conversation traces.
