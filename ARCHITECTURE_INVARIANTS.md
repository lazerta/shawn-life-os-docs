# Shawn Life OS — Architecture Invariants

Status: Long-term architecture invariants. These apply across releases unless explicitly changed by the user.

## INV-CORE-001 — Core owns semantics, not commodity implementation
The Life OS core owns canonical semantics, source/provenance/revision identity, publication/temporal semantics, idempotency/conflict behavior, privacy/egress boundaries, transaction/data-integrity rules, and acceptance criteria.

OCR engines, source clients, parsers, SDKs, model providers, document/media tooling, and similar commodity capabilities should remain external and connect through ports/adapters.

## INV-REUSE-001 — Open Source First / Reuse Before Build
Before implementing a non-core capability, search for a mature open-source project/library/tool first.

Required preference order:
`direct reuse -> configuration/extension -> thin adapter -> bounded custom gap code -> full custom implementation as last resort`

A custom replacement for an existing mature implementation requires a documented rejection reason.

## INV-DOC-001 — Document First
Behavior-changing implementation must be preceded by updates to the relevant requirements/specification/ADR/data-model/test expectations.

If implementation reveals that the design is wrong or incomplete, update the documentation before continuing implementation.

## INV-FACT-001 — Deterministic Fact Layer
Canonical factual records are produced only by deterministic, versioned code paths: parsers, typed adapters, schema validation, canonical mappings, normalization rules, identity/revision rules, constraints, and transactions.

Model/LLM/vision interpretation cannot directly create or overwrite canonical facts. Model output belongs in derived assertion/annotation/claim layers unless explicitly promoted through a separately documented deterministic or human-reviewed process.

## INV-STATE-001 — Source-Neutral / Model-Neutral Canonical State
The canonical layer must preserve conflicting evidence and must not silently privilege a source, vendor, model, or inferred narrative.

Any resolution from conflicting evidence must be explicit, versioned, attributable, reproducible, and auditable.

## INV-HIST-001 — Immutable + Append-Only History
Protected historical records are immutable after durable publication.

Protected history includes at minimum:
- SourceRevision;
- canonical fact/object versions;
- provenance links;
- AnnotationAssertion / Claim versions;
- publication ledger events;
- explicit supersession/resolution events.

Corrections and later source updates append new revisions/versions; they never rewrite prior history.

Database implementations must enforce immutability at the persistence boundary, not merely by application convention, except for explicitly documented controlled exceptions.

## INV-PROJ-001 — Projections are rebuildable
Current-state projections, latest-value pointers, indexes, caches, search/vector indexes, denormalized read models, and other materialized views may be mutable only if they are rebuildable from immutable history.

## INV-PURGE-001 — Privacy purge is a controlled exception
Immutability does not override an explicit privacy deletion requirement. When sensitive payload bytes must be physically deleted, preserve only a non-sensitive immutable deletion/tombstone record sufficient to prove that a purge occurred and why, without retaining the purged sensitive content.

## INV-PROV-001 — Provenance everywhere
Every durable fact and derived assertion must retain sufficient provenance to answer:
- source system and source instance;
- source record and source revision;
- mapper/producer/provider identity and version;
- transformation or deterministic mapping version;
- publication event;
- upstream fixture/dataset/project attribution when applicable.

## INV-TEST-001 — Spec-driven + Test-driven + Real-data-driven
Release-critical behavior must trace from documented invariant/requirement to acceptance criterion and executable evidence.

Preferred chain:
`requirement/invariant -> acceptance criterion -> real-data fixture/corpus -> deterministic/adversarial test -> observed runtime evidence -> PASS/FAIL`

When legally and operationally feasible, tests use real-world data from public internet datasets, upstream open-source fixtures, public benchmark corpora, public API samples, or user-authorized sources. Synthetic data supplements real data for rare boundaries, deterministic fault injection, fuzzing/property testing, and adversarial mutations.

Model output must never define its own ground truth.

## INV-COMMIT-001 — Code pre-commit regression gate
Documentation-only commits do not require the application regression suite.

Any commit that can affect executable behavior, production source, database migrations, build/CI/runtime configuration, packaging, or other runtime behavior must follow this sequence:
`document first -> summarize intended diff to user -> run full regression -> report PASS/FAIL -> commit only after PASS`

If documentation and code/runtime changes are bundled together, treat the commit as code-bearing.

## INV-EVIDENCE-001 — Evidence before PASS
No release, migration, validation gate, or consequential architecture claim is PASS without reproducible evidence. Insufficient evidence means NOT PROVEN.

## INV-DATA-001 — Internet/public real-data provenance
Public internet data may be used for testing when legally accessible and privacy-safe. Record the original source/repository, retrieval date, version/tag/commit when available, license/terms, attribution obligations, and every transformation/mutation applied.

Publicly accessible leaks, credentials, private dumps, or unlawfully exposed personal data are not acceptable test sources.

## INV-PLUGIN-001 — DataSources are pluggable and configurable
A DataSource is a first-class plugin boundary, not a hard-coded source-specific branch in canonical core logic.

Each DataSource plugin must declare a stable plugin ID/version, supported source-system type(s), acquisition channels, configuration schema, output semantic schema(s), capabilities, health/readiness behavior, secret references, and provenance contract.

A configured `SourceInstance` selects and configures a plugin/adapter declaratively. Installing, removing, enabling, disabling, or reconfiguring a plugin must not require changing canonical domain logic.

A single logical DataSource may expose multiple acquisition adapters. Distinct acquisition methods must preserve their own provenance while mapping into the same source-system/canonical semantics where appropriate.

Plugin failures must be isolated at the adapter/pipeline boundary and must not corrupt immutable history. Missing, disabled, unhealthy, or incompatible plugins fail explicitly and must not silently substitute another source or fabricate data.

## INV-SCHEMA-001 — Every pipeline-visible output has a versioned semantic schema
Every durable or pipeline-visible payload must declare a versioned semantic schema. Untyped or unversioned JSON is not a durable contract.

Schemas define semantic meaning as well as field shape, including units, timestamp semantics/precision/timezone interpretation, identity semantics, null/missing meaning, source-vs-derived status, and provenance requirements where applicable.

Processor and mapper boundaries must declare compatible input/output `SchemaRef`s. Semantic changes require an explicit schema and/or mapper version; historical records are not silently reinterpreted.

Preferred boundary chain:
`SourceEnvelope -> source-native normalized schema -> deterministic canonical schema -> derived/annotation schema -> projection/API schema`.

## INV-PIPE-001 — DataSources publish into typed pipelines
A configured DataSource instance may publish into one or more pipelines only through declared schema-compatible bindings.

Pipelines may be:
- deterministic fact pipelines;
- deterministic derived pipelines;
- explicitly probabilistic/model-based derived pipelines.

The DataSource/plugin owns acquisition and source-native normalization, not canonical resolution or downstream interpretation. Model-based processing cannot directly overwrite canonical facts.

## INV-UML-001 — Architecture diagrams are maintained as code
When architecture relationships, lifecycle, data lineage, state transitions, or cross-component flow are materially clarified by diagrams, maintain UML-style diagram-as-code alongside the normative documentation.

Mermaid is the preferred Markdown format for GitHub documentation because it is diffable, reviewable, version-controlled, and directly renderable.

At minimum, the DataSource/Pipeline architecture maintains:
- class/domain diagram;
- component/dependency diagram;
- ingestion sequence diagram;
- plugin lifecycle/state diagram;
- provenance/data-lineage diagram.

Diagrams do not replace normative prose, schemas, acceptance criteria, or tests. If a diagram and written invariant disagree, the inconsistency must be corrected before implementation. Architecture-changing work updates affected diagrams first under `INV-DOC-001`.