# Shawn Life OS — Engineering Principles

Status: Long-term project constitution. These principles apply across releases unless explicitly changed by the user.

## 1. Document First
Requirements, invariants, data semantics, adapter contracts, privacy boundaries, acceptance criteria, and test expectations are documented before implementation. Code is the execution of documented design, not the primary source of design intent.

Documentation-only commits do not require the application regression suite. Any commit that can affect executable behavior, migrations, build/CI/runtime configuration, or packaging is code-bearing and must follow the code pre-commit regression gate.

## 2. Open Source First
Before building any non-core capability, search for mature open-source projects, libraries, SDKs, parsers, benchmark corpora, and tools that already solve the problem. Prefer reuse over reinvention when license, maintenance, correctness, privacy, security, provenance, and deployment fit are acceptable.

Decision order:
1. direct reuse;
2. configuration or extension;
3. thin adapter/wrapper;
4. bounded custom code for Life OS-specific gaps;
5. full custom implementation only as a last resort.

If mature open-source code exists but is rejected, the rejection reason must be explicit and testable. Environment inconvenience alone is not sufficient justification.

## 3. Adapter / Port First
Commodity and source-specific capabilities remain outside the core and integrate through explicit ports/adapters. The core must not depend directly on vendor-specific representations.

Preferred dependency direction:
`external project/library/provider -> adapter -> Life OS port -> provenance/source model -> canonical core`

## 4. Code First / Model Last
Prefer deterministic code, standard algorithms, schemas, SQL, state machines, checksums, and mature libraries over model inference whenever they can solve the task reliably, cheaply, reproducibly, and audibly.

Models are reserved for genuinely ambiguous semantic tasks. Model output is derived assertion/annotation by default, not canonical fact.

## 5. Deterministic Fact Layer
Canonical factual records are produced by code-defined, versioned, reproducible mappings. The same accepted source revision + mapper version + configuration must produce the same canonical fact representation.

No hidden source ranking, plausibility judgment, model confidence, or narrative inference may directly create or overwrite facts.

## 6. Immutable + Append-Only History
Protected historical records are immutable and append-only. Corrections create new revisions/versions; prior history is not rewritten.

Protected history includes, at minimum:
- source revisions;
- canonical fact/object versions;
- provenance links;
- annotation/claim assertions;
- publication history.

Mutable state is limited to rebuildable projections, indexes, caches, and materialized current-state views. Privacy purge is a controlled exception for removal of sensitive payloads and must retain a non-sensitive tombstone/deletion ledger.

## 7. Source-Neutral / Model-Neutral Canonical State
Conflicting evidence is preserved rather than silently collapsed. Canonical storage must not privilege a source, vendor, model, or inferred narrative without an explicit, versioned resolution policy.

## 8. Provenance Everywhere
Every durable fact and derived assertion must retain sufficient provenance to answer where it came from, which source revision produced it, which mapper/producer/version created it, and how to reproduce it.

## 9. Rebuildable Projections
Current-state views, indexes, caches, and materialized projections may be mutable only when they are fully rebuildable from immutable history.

## 10. API First / Local First / Privacy First
The system is API-first and local-first. Privacy and egress boundaries are explicit architectural constraints, not post-processing concerns.

## 11. Spec-Driven + Test-Driven + Real-Data-Driven
Every meaningful behavior or invariant traces from specification to executable evidence:
`requirement/invariant -> acceptance criterion -> fixture/corpus -> deterministic/adversarial test -> runtime evidence -> PASS/FAIL`

Implementation work is accompanied by tests tied to documented requirements. For critical invariants, the expected/failing test is written before or alongside the implementation change.

Whenever legally and operationally feasible, validation uses real-world source data, public internet datasets, upstream project fixtures, public benchmark corpora, public API samples, or user-authorized records that preserve realistic distributions, timestamps, missingness, revisions, duplicates, ordering errors, encoding/layout quirks, and other source dirtiness.

Synthetic data is supplemental when suitable real data exists. Use it mainly for rare boundaries, deterministic fault injection, fuzzing/property tests, impossible-to-source safety cases, and adversarial mutations derived from real records.

Publicly accessible personal leaks, credentials, private dumps, or unlawfully exposed personal data are not acceptable test sources.

A release-critical PASS based only on happy-path synthetic examples is insufficient when representative real data is available.

## 12. Evidence Before PASS
No release, migration, validation gate, or architecture claim is PASS without reproducible evidence. If evidence is insufficient, status is NOT PROVEN rather than PASS.

## 13. Governance
Long-lived rules belong in durable governance documents. Versioned specs and ADRs describe how a specific release implements them. The Google Drive live handoff tracks current operational state and must not silently override these principles.

## 14. Pluggable + Configurable DataSources
DataSource support is a first-class plugin boundary. New sources are added through discoverable/configurable plugins and adapters rather than hard-coded source-specific branches in the canonical core.

A DataSource plugin declares its stable identity, version, acquisition channels, configuration schema, output semantic schemas, capabilities, health/readiness behavior, and provenance contract. Individual `SourceInstance` configurations enable/disable and bind plugins to processing pipelines declaratively.

One logical DataSource may expose multiple acquisition adapters. For example, WeChat may support screenshot, backup-database, export-file, and future API adapters without changing the canonical domain model.

Installing, removing, enabling, disabling, or reconfiguring a plugin must not require changing canonical domain logic.

## 15. Versioned Semantic Contracts + Diagram-as-Code
Every durable or pipeline-visible output declares a versioned semantic schema. A schema defines field meaning as well as shape; untyped/unversioned JSON is not a durable pipeline contract.

Architecture relationships, lifecycle, data flow, and state transitions should be documented with maintainable UML-style diagram-as-code when a diagram materially improves precision. Mermaid is the preferred Markdown format because it is diffable, reviewable, version-controlled, and rendered directly by GitHub.

Diagrams do not replace normative prose, schemas, acceptance criteria, or tests. When architecture relationships change, affected diagrams are updated before implementation under the Document First rule.

## 16. Shared Capability Infrastructure
Commodity capabilities that are useful across multiple DataSources or pipelines belong in shared infrastructure behind stable Life OS capability ports and configurable provider adapters. They must not be duplicated or owned by a single source plugin merely because that plugin is the first consumer.

Preferred dependency direction:
`DataSource / Pipeline -> Capability Port -> Provider Registry -> Provider Adapter -> open-source/external implementation`

Examples include OCR, speech-to-text, document parsing, media metadata extraction, image preprocessing, embeddings, and model inference.

OCR is the first concrete application of this rule. A WeChat screenshot adapter consumes `OcrPort`; it does not own PaddleOCR, RapidOCR, EasyOCR, or another engine. Other image/document sources reuse the same OCR infrastructure.

Provider selection is declarative and provenance-aware. Every durable provider result records the actual provider/adapter/model/runtime version and the source evidence it processed.

Probabilistic capability output such as OCR recognition is extracted/derived evidence by default, not canonical fact. Promotion into a source-native record or canonical fact must obey the relevant schema, deterministic mapping, validation, provenance, and uncertainty rules.

See `SHARED_CAPABILITY_INFRASTRUCTURE.md` for the normative provider/capability architecture.