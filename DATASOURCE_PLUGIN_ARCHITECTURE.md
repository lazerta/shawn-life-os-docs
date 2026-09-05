# Shawn Life OS — DataSource Plugin Architecture

Status: Long-term architecture design. This document defines the pluggable/configurable DataSource boundary and its UML/diagram-as-code representation.

## 1. Purpose

Life OS treats each external origin of evidence as a **DataSource** and each concrete way of acquiring from it as an **Acquisition Adapter**. DataSource support must be pluggable, declaratively configurable, schema-versioned, pipeline-compatible, and isolated from canonical core semantics.

The core owns semantic invariants, provenance, revision identity, canonical mapping contracts, publication, privacy boundaries, and immutable history. Source-specific acquisition and commodity parsing/OCR/API behavior remain outside the core and are normally implemented by reusing mature open-source projects behind thin adapters.

## 2. Required dependency direction

```mermaid
flowchart LR
    EXT[External system / open-source project / SDK]
    ADP[Acquisition Adapter]
    PORT[Life OS DataSource Port]
    SRC[Versioned Source Schema / SourceEnvelope]
    PIPE[Pipeline]
    MAP[Deterministic Canonical Mapper]
    HIST[(Immutable Canonical History)]
    DER[Derived Pipelines]
    PROJ[Rebuildable Projections]

    EXT --> ADP --> PORT --> SRC --> PIPE --> MAP --> HIST
    HIST --> DER
    HIST --> PROJ
    DER --> PROJ
```

The canonical core must not directly depend on vendor-specific representations.

## 3. UML class/domain model

```mermaid
classDiagram
    class DataSourcePlugin {
      +PluginId pluginId
      +SemVer pluginVersion
      +Set~SourceSystemType~ supportedSourceSystems
      +Set~AcquisitionChannel~ acquisitionChannels
      +SchemaRef configurationSchema
      +Set~SchemaRef~ outputSchemas
      +PluginCapabilities capabilities
      +healthCheck()
      +createAdapter(config)
    }

    class SourceInstanceConfig {
      +SourceInstanceId sourceInstanceId
      +PluginId pluginId
      +SemVer pluginVersion
      +boolean enabled
      +AcquisitionChannel acquisitionChannel
      +SchemaRef configSchema
      +ConfigVersion configVersion
      +SecretRef[] secretRefs
      +PipelineBinding[] pipelineBindings
    }

    class AcquisitionAdapter {
      <<interface>>
      +SchemaRef outputSchema()
      +acquire(context) SourceEnvelope[]
    }

    class SourceEnvelope {
      +SchemaRef schema
      +SourceSystemId sourceSystem
      +SourceInstanceId sourceInstance
      +AcquisitionChannel acquisitionChannel
      +ContentRef payloadRef
      +Instant capturedAt
      +Provenance provenance
    }

    class SchemaRef {
      +String schemaId
      +SemVer version
    }

    class Pipeline {
      +PipelineId pipelineId
      +SchemaRef inputSchema
      +Processor[] processors
      +SchemaRef outputSchema
    }

    class Processor {
      <<interface>>
      +SchemaRef inputSchema()
      +SchemaRef outputSchema()
      +process(input, context)
    }

    class CanonicalMapper {
      <<interface>>
      +SchemaRef inputSchema()
      +SchemaRef canonicalSchema()
      +MapperVersion mapperVersion()
      +map(sourceRevision)
    }

    class SourceRecord {
      +SourceRecordId id
      +SourceInstanceId sourceInstanceId
      +String sourceRecordKey
    }

    class SourceRevision {
      +SourceRevisionId id
      +SourceRecordId sourceRecordId
      +SourceRevisionId predecessorRevisionId
      +SchemaRef sourceSchema
      +String payloadFingerprint
    }

    class CanonicalObjectVersion {
      +ObjectId objectId
      +VersionId versionId
      +SchemaRef canonicalSchema
      +MapperVersion mapperVersion
      +PublishedSeq publishedSeq
    }

    class AnnotationAssertion {
      +AnnotationId annotationId
      +TargetVersionId targetVersionId
      +ProducerRef producer
      +SchemaRef annotationSchema
      +AnnotationId supersedesAnnotationId
    }

    DataSourcePlugin "1" --> "0..*" SourceInstanceConfig : configures
    DataSourcePlugin "1" --> "1..*" AcquisitionAdapter : exposes
    AcquisitionAdapter --> SourceEnvelope : emits
    SourceEnvelope --> SchemaRef : declares
    SourceInstanceConfig --> Pipeline : binds
    Pipeline "1" *-- "1..*" Processor : contains
    Processor --> SchemaRef : typed boundary
    Pipeline --> CanonicalMapper : may invoke
    CanonicalMapper --> SourceRevision : maps from
    SourceRecord "1" --> "1..*" SourceRevision : revisions
    SourceRevision "1..*" --> "0..*" CanonicalObjectVersion : provenance
    CanonicalObjectVersion "1" --> "0..*" AnnotationAssertion : target
```

## 4. DataSource plugin contract

Every plugin must declare, at minimum:

- stable `pluginId` and plugin version;
- supported source-system type(s);
- acquisition channel(s);
- configuration schema and version;
- output semantic schema(s) and versions;
- required capabilities/runtime prerequisites;
- health/readiness behavior;
- secret references, never embedded secret values;
- provenance metadata sufficient to reproduce acquisition;
- compatible pipeline bindings;
- explicit failure behavior.

A plugin may expose multiple acquisition adapters. For example, a WeChat DataSource may support screenshot, backup database, export file, and a future API adapter while retaining the same logical DataSource identity and downstream canonical model.

## 5. Configuration model

DataSource instances are declaratively configured. Core domain code must not be edited merely to add, enable, disable, or reconfigure a source plugin.

Configuration may control:

- enabled/disabled state;
- plugin and adapter selection;
- source-instance identity;
- import/poll/event mode;
- path/endpoint/device/account references;
- credentials via `SecretRef`;
- pipeline bindings;
- privacy/egress policy;
- retention policy;
- resource/rate limits;
- plugin-specific options.

Configuration is validated against the plugin-declared schema before activation. Configuration that changes semantics or canonical mapping behavior must be versioned and referenced in provenance so historical outputs remain reproducible.

## 6. Versioned semantic schema rule

Every pipeline-visible output declares a versioned semantic schema. Untyped or unversioned JSON is not a durable contract.

Typical boundaries:

`SourceEnvelope -> source-native normalized schema -> deterministic canonical schema -> derived/annotation schema -> projection/API schema`

A schema defines **meaning**, not only JSON shape. For example, timestamp semantics must specify origin, precision, timezone interpretation, and whether a timestamp was source-reported, observed, inferred, or normalized.

Schema changes are explicit versions. A semantic change is not silently applied to historical records; it requires a new schema and/or mapper version.

## 7. Ingestion sequence UML

```mermaid
sequenceDiagram
    participant REG as Plugin Registry
    participant CFG as Config Service
    participant PLG as DataSource Plugin
    participant ADP as Acquisition Adapter
    participant VAL as Schema Validator
    participant ING as Ingestion Core
    participant MAP as Canonical Mapper
    participant DB as Immutable History
    participant DP as Derived Pipelines

    REG->>PLG: discover plugin metadata/capabilities
    CFG->>VAL: validate SourceInstanceConfig against plugin config schema
    VAL-->>CFG: valid
    CFG->>PLG: activate configured instance
    PLG->>ADP: create adapter(config, secret refs)
    ADP->>ING: SourceEnvelope@version
    ING->>VAL: validate envelope/source schema
    VAL-->>ING: valid
    ING->>DB: append SourceRecord / SourceRevision
    ING->>MAP: map accepted SourceRevision + mapperVersion + configVersion
    MAP->>DB: append CanonicalObjectVersion + provenance + publication
    DB-->>DP: publish immutable version reference
    DP->>DB: append derived assertion/annotation when durable
```

No model step is permitted in the deterministic fact path unless the output remains explicitly derived and does not directly create/overwrite canonical fact state.

## 8. Plugin lifecycle UML

```mermaid
stateDiagram-v2
    [*] --> DISCOVERED
    DISCOVERED --> CONFIGURED: valid config supplied
    CONFIGURED --> VALIDATED: config/schema/capabilities pass
    VALIDATED --> ACTIVE: activate
    ACTIVE --> DEGRADED: health/readiness failure
    DEGRADED --> ACTIVE: recovered
    ACTIVE --> DISABLED: operator/config disables
    DEGRADED --> DISABLED: disable
    DISABLED --> VALIDATED: re-enable + revalidate
    CONFIGURED --> INVALID: config/schema incompatible
    INVALID --> CONFIGURED: corrected config
    DISABLED --> REMOVED: uninstall/remove
    DISCOVERED --> REMOVED: uninstall/remove
    REMOVED --> [*]
```

Failures are explicit and fail closed. A missing, unhealthy, disabled, or incompatible plugin must not silently substitute another source or fabricate evidence.

## 9. Data lineage / provenance UML

```mermaid
flowchart TD
    A[External Evidence]
    B[Open-source Parser / SDK / OCR\nproject + version/commit]
    C[Life OS Acquisition Adapter\nadapter version]
    D[SourceEnvelope\nschema version]
    E[SourceRecord]
    F[SourceRevision\npayload fingerprint + predecessor]
    G[Canonical Mapper\nmapper version + config version]
    H[CanonicalObjectVersion\nimmutable]
    I[Annotation / Claim Producer\nmodel/rule version]
    J[AnnotationAssertion / Claim\nimmutable derived record]

    A --> B --> C --> D --> E --> F --> G --> H
    H --> I --> J
```

The system must be able to reconstruct the path from any durable canonical/derived record back to source evidence and the exact parser/adapter/schema/mapper/configuration/producer versions involved.

## 10. Pipeline fan-out

A single configured DataSource instance may feed multiple pipelines as long as each binding is schema-compatible and provenance is retained.

```mermaid
flowchart LR
    DS[Configured DataSource Instance]
    ENV[Versioned SourceEnvelope]
    FACT[Deterministic Fact Pipeline]
    ATT[Attachment Pipeline]
    IDX[Search/Index Pipeline]
    SEM[Semantic/Model Pipeline]
    HIST[(Immutable Canonical History)]
    ANN[(Immutable Derived Assertions)]
    PROJ[Rebuildable Projection]

    DS --> ENV
    ENV --> FACT --> HIST
    ENV --> ATT
    HIST --> IDX --> PROJ
    HIST --> SEM --> ANN
    ANN --> PROJ
```

The DataSource plugin owns acquisition/source-native normalization. It does **not** own canonical resolution or downstream interpretation.

## 11. WeChat as the first real plugin slice

WeChat is a DataSource, not a screenshot feature.

Initial adapter path:

`WeChat DataSource -> screenshot adapter -> WhoChat parser/replay + mature OCR provider -> wechat.normalized-chat@1.x -> Life OS ingestion pipeline -> immutable source revisions -> deterministic canonical message mapping`

Possible future adapters:

- backup DB adapter;
- export-file adapter;
- other lawful/local acquisition methods;
- future official API adapter if available.

All adapters retain the same logical source-system identity while recording their distinct acquisition channel and provenance.

## 12. Open-source-first rule

A DataSource plugin should normally wrap mature open-source projects/SDKs/parsers rather than duplicate their internals. Life OS-owned code should focus on:

- stable port/adapter boundary;
- configuration and schema validation;
- provenance and source identity;
- privacy/egress boundary;
- deterministic canonical mapping;
- pipeline integration;
- reproducible tests and real-data validation.

Any broader custom implementation requires a documented reuse rejection decision.

## 13. Diagram governance

These diagrams are normative architecture aids and must remain consistent with the written invariants. They do not replace schemas, requirements, acceptance criteria, or executable tests.

Use diagram-as-code so architecture changes are diffable and reviewable. Update affected diagrams before implementing a relationship/lifecycle/dataflow change under the Document First rule.