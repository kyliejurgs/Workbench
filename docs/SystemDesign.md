# Workbench

## System Design Document

**Author:** Kylie Jurgensen  
**Status:** Pre-implementation architecture baseline

---

## About this Document

This System Design Document describes how Workbench is designed and implemented from an engineering perspective.

The companion `WorkbenchV1Specs.md` defines the intended Workbench v1 product, product semantics, object model, behavior, and high-level architectural requirements. It is authoritative for what Workbench means.

This document defines how those requirements are implemented.

If the v1 specification and this System Design disagree about product meaning or behavior, the v1 specification takes priority unless the specification is intentionally changed.

Detailed implementation decisions that do not affect architectural boundaries may be made during development without requiring this document to define them in advance.

---

# Part I — Architecture Foundations

## 1. Purpose and Scope

Workbench is a local-native desktop application for building, organizing, analyzing, and visualizing structured data.

The architecture supports:

- Spreadsheet-like structured-data interaction
- Tables, Fields, Records, and Views
- References and relationships
- Formulas and Summaries
- Queries and Dashboards
- Embedded Files
- Durable history, undo, redo, and recovery
- Automatic persistence
- Large local datasets
- Portable self-contained `.wkbn` artifacts
- Offline operation
- Cross-platform desktop execution

Workbench is designed around the principle:

> **Feel like a spreadsheet; behave like structured data underneath.**

The grid is an important presentation and interaction surface. It does not define the underlying data model.

---

## 2. Governing Engineering Principles

### 2.1 Product Semantics Are Authoritative

Workbench product semantics are defined by the v1 specification and application-owned domain model.

Frameworks, SQLite, grid libraries, desktop runtimes, file formats, and third-party libraries implement Workbench behavior. They do not define it.

When implementation convenience conflicts with Workbench semantics, Workbench semantics take priority.

### 2.2 Local-Native Operation

Core Workbench functionality operates entirely against local workbench state.

Opening, editing, querying, calculating, searching, saving, and recovering a workbench must not require:

- network connectivity;
- a cloud service;
- an account;
- a remote database; or
- a synchronization server.

Future cloud functionality may extend the local architecture but must not retroactively make local operation dependent on cloud infrastructure.

### 2.3 Domain Operations Represent Meaningful Changes

Persistent changes are represented as coherent domain operations.

Examples include:

- updating a Field value;
- creating a Record;
- deleting Records;
- creating a Field;
- changing a Field type;
- moving a resource;
- changing a Formula;
- publishing an imported Table.

Domain operations are the boundary for:

- validation;
- dependency effects;
- transactions;
- persistence;
- history;
- undo and redo;
- invalidation; and
- recovery behavior.

### 2.4 Modular Desktop Monolith

Workbench is architected as a modular desktop monolith.

Processes, worker threads, runtime workers, or other execution contexts may be introduced for responsiveness, isolation, or performance. Their existence does not divide domain authority.

Workbench should not reproduce distributed-system architecture inside a local desktop application without a demonstrated need.

### 2.5 Application-Owned Product Behavior

Workbench owns semantics that define the product, including:

- Tables;
- Fields;
- Records;
- Views;
- References;
- Formulas;
- Summaries;
- Queries;
- dependencies;
- history;
- Grid behavior; and
- structured-data interoperability.

Libraries may provide commodity capability but must not silently redefine these semantics.

### 2.6 Infrastructure Must Solve a Real Problem

Infrastructure is introduced when it solves a measured or clearly established problem.

Workbench does not add:

- processes;
- workers;
- caches;
- abstraction layers;
- synchronization machinery; or
- native components

solely because they may be useful in the future.

### 2.7 Protect User Data

Durability and recoverability take priority over implementation convenience.

Failed edits, migrations, imports, cancellations, crashes, updates, or maintenance operations must not knowingly leave authoritative workbench state partially applied or silently discard recoverable user data.

### 2.8 Localize Failure

Failure should be contained to the smallest practical operation, resource, or subsystem.

A failure in derived state should not make unrelated authoritative data unusable where safe recovery is possible.

Artifact-integrity concerns may override failure localization when continued operation would risk user data.

### 2.9 Autosave Is the Persistence Model

Autosave is not a periodic export of authoritative UI state.

A successful persistent domain operation is durably committed as part of completing that operation.

React does not hold an independent unsaved authoritative copy of the workbench.

Optimistic presentation may exist, but persistent Workbench Core state remains authoritative.

### 2.10 Accessibility Is Architectural

Accessibility requirements apply to the application shell, Grid, editors, dialogs, dashboards, and other interaction surfaces.

Rendering technology must not be selected in a way that makes accessibility an afterthought.

### 2.11 Structured-Data Semantics First

Spreadsheet interoperability is first-class, but external spreadsheet formats do not define Workbench's internal semantics.

CSV, XLSX, clipboard, and similar formats are translated through explicit Workbench interoperability rules.

### 2.12 Configuration Is Centralized

Configurable policy and implementation tuning are expressed through defined configuration boundaries rather than scattered constants or behavioral branches.

### 2.13 Live Dependencies Stay Within a Workbench

Live dependencies do not cross `.wkbn` boundaries in v1.

Copying or importing resources between workbenches creates or remaps local identities rather than creating cross-workbench live references.

### 2.14 Stable Identity Drives Relationships

Persisted domain objects use stable internal UUIDs.

Names, positions, paths, Collection membership, and presentation order do not define identity.

---

## 3. System Architecture

Workbench uses the following responsibility architecture:

```text
Operating System
      │
      ▼
Desktop Runtime / Native Shell
      │
      ▼
React UI
├── Application Shell
├── Resource Surfaces
├── Grid
└── Presentation / Interaction State
      │
      ▼
Workbench Core API
      │
      ▼
Workbench Core
├── Application Services
├── Domain Operations
├── Domain Systems
├── Expression Engine
├── Dependency System
├── History
└── Search
      │
      ├───────────────┐
      ▼               ▼
Data / Query      Background Execution
      │               │
      └───────┬───────┘
              ▼
            SQLite
              │
              ▼
            .wkbn
```

These are responsibility boundaries, not mandatory process boundaries.

Physical process, thread, and worker topology may evolve without changing the logical architecture.

---

## 4. Major Responsibility Boundaries

### 4.1 Desktop Runtime / Native Shell

The desktop runtime owns operating-system integration, including:

- application lifecycle;
- window lifecycle;
- native file dialogs;
- `.wkbn` file associations;
- operating-system open-with behavior;
- drag and drop from the operating system;
- native clipboard integration where required;
- application packaging;
- distribution;
- update integration; and
- supervision of runtime execution contexts.

MōBrowser-specific behavior is isolated behind application-owned capabilities where practical.

Workbench domain behavior does not belong in the native shell.

### 4.2 React UI

React owns presentation and interaction state.

Examples include:

- tabs;
- panels;
- selection presentation;
- focus;
- menus;
- dialogs;
- scroll position;
- temporary editor state;
- drag state;
- command presentation;
- bounded displayed data.

React does not own authoritative Workbench domain state.

React does not directly access SQLite or the internal `.wkbn` schema.

### 4.3 Workbench Core

Workbench Core owns authoritative product behavior.

This includes:

- Tables;
- Fields;
- Records;
- Views;
- Queries;
- Dashboards;
- References;
- Formulas;
- Summaries;
- Files;
- dependencies;
- validation;
- history;
- persistent mutations;
- application-level integrity.

### 4.4 Data / Query Layer

The Data / Query layer owns:

- repository access;
- bounded retrieval;
- filtering;
- sorting;
- grouping;
- aggregation;
- query planning;
- query execution;
- indexing;
- pagination/window retrieval;
- SQLite translation.

SQL is an implementation detail below this boundary.

### 4.5 Background Execution

Background Execution handles expensive work that should not block interactive UI execution.

Potential workloads include:

- large imports and exports;
- formula recalculation;
- Summary recalculation;
- query execution;
- index construction;
- search indexing;
- file processing;
- integrity checks;
- maintenance.

Background Execution is an execution mechanism, not an independent domain authority.

---

# Part II — Desktop Application and UI

## 5. Desktop Runtime

### 5.1 Preferred Runtime

Workbench v1 targets **MōBrowser 2.x** as its preferred desktop runtime.

Electron remains the primary fallback if validation identifies a concrete MōBrowser limitation that materially conflicts with Workbench requirements.

Undocumented behavior alone is not treated as evidence of a limitation.

### 5.2 Runtime Independence

Core domain concepts must not depend on MōBrowser-specific APIs.

Runtime-specific behavior is isolated so that replacing the desktop runtime does not require redesigning:

- the Workbench domain;
- persistence;
- Query Plans;
- history;
- dependency semantics; or
- Grid semantics.

### 5.3 Physical Execution Topology

The architecture does not prescribe a fixed physical process topology.

The initial implementation should use the simplest topology that satisfies:

- responsiveness;
- safety;
- native integration;
- SQLite requirements; and
- background workload requirements.

Threads, workers, or processes are introduced only when validated workloads justify them.

---

## 6. Frontend Architecture

### 6.1 Framework

Workbench uses:

- React;
- TypeScript; and
- a modern frontend build tool compatible with the selected runtime.

React is the UI framework.

React is not the:

- persistence layer;
- query engine;
- formula engine;
- dependency system; or
- domain authority.

### 6.2 Application Shell

Application-level layout belongs to React.

The shell includes:

- title/menu areas;
- command interface;
- Activity Bar;
- left panel;
- right panel;
- bottom panel;
- tabbed work area;
- status bar.

Persistent domain presentation configuration belongs to Workbench Core when it represents saved workbench state.

### 6.3 State Management

UI state and authoritative domain state are distinct.

React may cache bounded data required for presentation, but the renderer must never assume ownership of entire large Tables.

---

## 7. Grid Architecture

### 7.1 Application-Owned Semantics

Workbench owns Grid semantics, including:

- navigation;
- selection;
- editing behavior;
- clipboard behavior;
- Field behavior;
- command interpretation;
- data retrieval semantics.

### 7.2 Grid Library Policy

Workbench may use a third-party high-performance grid library for rendering and interaction.

Any grid library operates behind an application-owned Grid Adapter.

The library must not become authoritative for:

- Workbench data;
- Formulas;
- Queries;
- dependencies;
- history;
- persistence; or
- domain identity.

### 7.3 Rendering Technology

The final Grid renderer is intentionally not fixed by this architecture.

DOM, Canvas, hybrid rendering, or a library-owned rendering implementation may be used if it satisfies Workbench requirements.

Renderer selection is a validation decision.

### 7.4 Bounded Data

The Grid requests bounded windows.

Conceptually:

```text
Grid
  │
  ▼
Grid Adapter
  │
  ▼
Workbench View / Query Plan
  │
  ▼
Data / Query Layer
  │
  ▼
SQLite
```

A large Table is never materialized in React merely because it is open in the Grid.

### 7.5 Accessibility

Grid accessibility semantics must remain application-controlled even if visual rendering is delegated to a library.

---

# Part III — Workbench Core

## 8. Domain Operations

Persistent mutations flow through Workbench Core.

A typical mutation follows:

```text
UI Intent
   ↓
Domain Operation
   ↓
Validation
   ↓
Dependency Analysis
   ↓
Transaction
   ├── authoritative mutation
   ├── history
   └── dependency invalidation
   ↓
Commit
```

Domain operations provide the semantic boundary between UI intent and persistence mechanics.

---

## 9. Expression Engine

Workbench retains an application-owned typed expression engine.

The expression engine defines Formula and other expression semantics independently of SQLite syntax.

Expression consumers may include:

- Formula Fields;
- Query expressions;
- filtering;
- conditional behavior;
- Summaries;
- future calculated systems.

SQLite may execute translated portions of expressions where semantics are equivalent.

SQLite behavior does not redefine Workbench expression semantics.

Decimal-sensitive operations must preserve Workbench numeric semantics rather than silently adopting inappropriate binary floating-point behavior.

---

## 10. Formula and Summary Derived State

Formula and Summary definitions are authoritative.

Calculated results are derived state.

Derived results may be:

- calculated on demand;
- cached;
- materialized;
- indexed; or
- rebuilt

depending on performance requirements.

Materialization strategy must not change product semantics.

A derived result may be discarded and regenerated from authoritative state when safe.

---

## 11. Dependency Architecture

### 11.1 Source-Owned Dependencies

Canonical dependency information belongs to the source definition.

Examples:

- a Formula owns its Field references;
- a Summary owns its source configuration;
- a Query owns its sources and expressions;
- a Widget owns its data-source configuration.

### 11.2 Dependency Index

Workbench maintains a derived dependency index for efficient reverse traversal and impact analysis.

The index is rebuildable from authoritative definitions.

### 11.3 Invalidation

When authoritative source state changes, affected derived state is explicitly invalidated.

Workbench must not silently treat known-stale derived results as current.

### 11.4 Recalculation

Derived recalculation may occur:

- synchronously when inexpensive; or
- through Background Execution when expensive.

Conceptually:

```text
Authoritative Change
        ↓
Commit + Invalidate
        ↓
   Derived state
      INVALID
        ↓
Recalculation
        ↓
      VALID
```

Validity must be explicitly knowable.

The exact generation/version representation is an implementation detail.

### 11.5 Cycles

Dependency cycles are detected through application-owned domain semantics.

Background execution must not be relied upon to accidentally discover cycles through runaway recalculation.

---

# Part IV — Persistence and `.wkbn`

## 12. Native Artifact

A `.wkbn` file is the live, versioned, self-contained Workbench artifact.

It is not an export package.

It is not a directory bundle.

It is not a temporary representation assembled from distributed authoritative systems.

Conceptually:

```text
Workbench state = .wkbn
```

In v1, the physical `.wkbn` artifact is implemented as a SQLite database using an application-owned schema.

SQLite is an implementation detail.

The `.wkbn` file remains a Workbench document from the user's perspective.

---

## 13. SQLite Role

SQLite is Workbench's primary:

- structured-data persistence engine;
- transactional storage engine;
- local set-oriented query engine;
- indexing engine;
- full-text-search foundation where appropriate.

Workbench owns domain and query semantics.

SQLite does not define the product model.

React never accesses SQLite directly.

The internal SQLite schema is private and is not a supported interoperability API.

---

## 14. Persistence Model

### 14.1 Logical Model

Workbench retains the conceptual value model:

```text
(recordId, fieldId) → value
```

The physical schema does not need to mirror that conceptual model.

### 14.2 Hybrid Relational Storage

Workbench uses a hybrid relational SQLite persistence model.

Ordinary scalar stored Fields use table-oriented physical storage optimized for relational retrieval and query execution.

Specialized structures are used for data that does not naturally fit scalar columns, including as appropriate:

- multi-value relationships;
- References;
- metadata;
- dependencies;
- history;
- Files;
- derived indexes;
- specialized Field structures.

### 14.3 Stable Physical Identity

Physical persistence uses stable internal identity rather than user-facing names.

Renaming a Field or Table must not require identity replacement.

User-facing names must not become persistence identity.

### 14.4 Logical Independence

A Workbench Table is not semantically equivalent to a SQLite table.

The persistence mapping remains below the repository/data boundary so future `.wkbn` format versions may evolve physical representation without changing product semantics.

---

## 15. Embedded Files

Files remain logically separate from structured storage even when physically stored inside the same SQLite artifact.

Conceptually:

```text
Workbench Core
      │
      ├── Structured Repository → SQLite structured storage
      │
      └── File Store            → SQLite binary storage
```

The File Store abstraction owns binary persistence.

The exact binary chunking and storage strategy is an implementation decision.

This boundary permits future artifact versions to change binary representation without changing File semantics.

---

## 16. Transactions and Concurrency

### 16.1 Single Mutation Authority

Each open workbench has one authoritative mutation stream.

Persistent mutations are serialized through Workbench Core and its transaction boundary.

### 16.2 Concurrent Reads

Bounded reads may execute concurrently using appropriate SQLite read connections where supported and beneficial.

### 16.3 Background Work

Background execution may perform:

- reads;
- preparation;
- staging;
- calculation; and
- other non-authoritative work

concurrently.

It may not independently bypass domain mutation authority.

### 16.4 Long-Running Work

Long-running work should not hold the authoritative write transaction for its entire duration when avoidable.

Appropriate patterns include:

- staging;
- batching;
- preparation outside the publication transaction;
- atomic publication;
- rebuildable derived work.

Exact SQLite connection counts, journaling modes, transaction modes, busy behavior, and thread affinity remain implementation decisions.

---

# Part V — Query and Search

## 17. Query Architecture

### 17.1 Query Plan

Workbench retains an application-owned typed Query Plan.

The Query Plan describes query semantics without making raw SQL the product model.

### 17.2 Hybrid Execution

Query execution is hybrid.

SQLite-compatible operations are pushed into SQLite where Workbench semantics can be preserved.

Workbench-specific evaluation may occur in application-owned systems where required.

Conceptually:

```text
Query
  ↓
Typed Query Plan
  ↓
Query Planner
  ├── SQLite execution
  └── Workbench semantic evaluation
  ↓
Bounded result
```

### 17.3 No Unbounded Application Fallback

Application-side execution must not silently respond to unsupported query behavior by loading an unbounded large dataset into memory.

The planner must instead use:

- bounded execution;
- SQLite-compatible translation;
- materialization;
- streaming;
- specialized execution; or
- a clear product limitation.

### 17.4 Query Results

UI consumers receive bounded result windows or streams appropriate to the use case.

---

## 18. Search

Search is distinct from structured Query filtering.

Workbench provides an application-owned Search Service.

SQLite capabilities, including full-text search where appropriate, provide execution and indexing.

Workbench owns:

- searchable-resource semantics;
- ranking behavior;
- identity;
- result categorization;
- navigation.

Search indexes are derived and rebuildable.

Search returns bounded results.

File metadata is searchable where appropriate. Full file-content extraction and indexing is an extensible capability rather than a foundational requirement.

---

# Part VI — History, Recovery, and Artifact Lifecycle

## 19. History, Undo, and Redo

### 19.1 Semantic History

Workbench history is domain-operation based.

History represents meaningful Workbench changes rather than raw SQL statements or SQLite page changes.

### 19.2 Atomic History

Persistent domain mutations atomically record the history metadata and recovery information required by that operation.

Normal mutations must not commit authoritative state and then separately attempt to write required history.

### 19.3 Undo Representation

The physical undo representation may vary by operation.

Strategies may include:

- inverse data;
- preserved before-state;
- snapshots;
- references to recovery structures;
- specialized restoration.

No requirement exists that every operation serialize a simple inverse command.

### 19.4 Undo as Domain Behavior

Undo and redo execute through Workbench Core.

They do not directly manipulate SQLite outside domain semantics.

### 19.5 History Retention

History supports retention and compaction.

Workbench does not require indefinite physical preservation of every historical representation.

Exact retention policy is a product/implementation decision to be finalized separately.

---

## 20. Crash Recovery

SQLite transactions provide the primary atomicity mechanism for normal persistent operations.

Normally:

```text
Committed operation   → present after recovery
Uncommitted operation → absent after recovery
```

Workbench recovery additionally handles application-level concerns such as:

- interrupted long-running work;
- incomplete derived recalculation;
- failed maintenance;
- migration failure;
- invalid derived state;
- artifact integrity concerns.

Rebuildable derived state should be discarded and regenerated rather than treated as authoritative recovery state where practical.

---

## 21. `.wkbn` Lifecycle and Maintenance

Stored state is classified as:

### 21.1 Authoritative State

State that cannot be casually discarded, including current domain data and retained history.

### 21.2 Rebuildable Derived State

Examples include:

- dependency indexes;
- Formula materializations;
- search indexes;
- query caches;
- other performance accelerators.

### 21.3 Reclaimable Obsolete State

Examples may include:

- history outside retention;
- abandoned staging state;
- deleted physical Field storage;
- obsolete snapshots;
- orphaned File chunks;
- superseded materializations.

### 21.4 Maintenance

Normal domain operations prioritize responsiveness and durability rather than immediate physical reclamation.

Maintenance may:

- rebuild derived state;
- reclaim obsolete storage;
- enforce history retention;
- check integrity;
- compact storage;
- clean staging state.

Maintenance should be automatic where practical.

Significant maintenance exposes progress and supports safe cancellation where appropriate.

---

## 22. Integrity

Workbench integrity has two levels.

### 22.1 SQLite Integrity

Physical database integrity is validated using appropriate SQLite mechanisms.

### 22.2 Workbench Semantic Integrity

Workbench also validates application invariants such as:

- valid object identity;
- valid metadata structure;
- dependency consistency;
- schema/format compatibility;
- File metadata/content consistency;
- required ownership relationships.

A rebuildable derived index being invalid does not necessarily make authoritative workbench state invalid.

---

## 23. Format Versioning and Migration

`.wkbn` is a versioned application-owned artifact.

Opening an older supported format may require migration.

Migration must preserve recoverability.

A failed migration must not knowingly destroy the last usable state of a workbench.

Exact migration techniques may vary based on migration size and risk.

The internal SQLite schema is private and may evolve between Workbench format versions.

---

# Part VII — Import, Export, and Interoperability

## 24. Import Architecture

Imports are streaming, bounded, application-owned, and format-adapter based.

Conceptually:

```text
External Source
      ↓
Format Adapter
      ↓
Streaming Parser
      ↓
Workbench Conversion / Validation
      ↓
Non-authoritative Staging
      ↓
Atomic Publication
      ↓
Authoritative Resource
```

Large imports must not require complete source materialization in application memory.

Staging may be written incrementally.

Cancellation or failure before publication leaves authoritative Workbench state unchanged.

External parsing libraries do not define Workbench type or conversion semantics.

---

## 25. Export Architecture

Exports operate from Workbench semantics and stream bounded results to format adapters.

Conceptually:

```text
Workbench View / Query
        ↓
Bounded result stream
        ↓
Format Adapter
        ↓
External file
```

Exporters do not depend on direct access to the private SQLite schema.

---

## 26. Clipboard and Spreadsheet Interoperability

Clipboard, CSV, XLSX, and other spreadsheet-oriented formats are translated through explicit Workbench interoperability rules.

Internal identity and structure may be preserved through application-owned clipboard formats where useful.

External representations are compatibility surfaces, not canonical domain models.

---

# Part VIII — Background Execution

## 27. Background Execution Model

Background execution exists to protect interactive responsiveness.

It is not an independent domain authority.

Long-running tasks should provide:

- progress where meaningful;
- cancellation where safe;
- localized failure;
- durable recovery behavior where required.

Background work that produces authoritative changes publishes those changes through Workbench Core.

### 27.1 Recalculation

Large dependency recalculation may execute in the background after authoritative mutation and invalidation have committed.

### 27.2 Query Work

Expensive query work may use background execution where necessary.

### 27.3 Import and Export

Large imports and exports execute incrementally without blocking interactive UI execution.

### 27.4 Maintenance

Index construction, integrity work, and artifact maintenance may use background execution.

---

# Part IX — Performance and Scale

## 28. Scale Targets

Workbench v1 targets:

- **1 million Records:** routine;
- **10 million Records:** primary engineering target;
- **50 million Records:** stress target;
- **100 million+ Records:** best effort; architecture should not fundamentally prevent it.

These targets refer to storage and data-processing capability, not rendering all Records simultaneously.

---

## 29. Bounded Data Principle

Frontend components operate on bounded data.

The UI must not own entire large Tables.

This applies to:

- Grid retrieval;
- Query results;
- Search results;
- export pipelines;
- background processing;
- preview surfaces.

Memory consumption should generally depend on active working-set size rather than total workbench size.

---

## 30. Performance Strategy

Performance should primarily come from:

- relational SQLite storage;
- appropriate indexing;
- query pushdown;
- bounded retrieval;
- streaming;
- virtualization;
- derived materialization where justified;
- background execution;
- measured optimization.

Architectural complexity is introduced based on profiling and validation rather than speculation.

---

# Part X — Quality and Engineering Constraints

## 31. Testing Strategy

Workbench testing should include:

### 31.1 Domain Tests

Test product semantics independently of UI and persistence implementation where practical.

### 31.2 Persistence Tests

Test:

- transaction atomicity;
- migrations;
- crash recovery;
- history durability;
- schema evolution;
- large-data behavior.

### 31.3 Query Conformance Tests

Equivalent Workbench semantics must produce equivalent results regardless of whether execution occurs through SQLite translation or application-owned evaluation.

### 31.4 Dependency Tests

Test:

- invalidation;
- recalculation;
- cycle detection;
- rebuildable dependency indexes;
- interrupted recalculation.

### 31.5 Interoperability Tests

Test CSV, XLSX, clipboard, and other supported formats against explicit Workbench conversion semantics.

### 31.6 Accessibility Tests

Accessibility testing is required for ordinary React UI and Grid interaction.

### 31.7 Scale Tests

Performance tests should include representative datasets at:

- 1M;
- 10M; and
- stress-scale sizes where practical.

---

## 32. Dependency Philosophy

Workbench prefers application-owned behavior for product-defining semantics and focused third-party libraries for commodity capabilities.

Dependencies should be selected based on:

- maintainability;
- performance;
- licensing;
- cross-platform compatibility;
- replaceability;
- accessibility;
- security;
- runtime compatibility.

A dependency must not become authoritative for Workbench semantics merely because it is convenient.

---

## 33. Explicitly Excluded v1 Infrastructure

Workbench v1 does not require:

- cloud authority;
- server-side application Core;
- REST as an internal application boundary;
- WebSocket coordination;
- PostgreSQL;
- IndexedDB as authoritative persistence;
- RabbitMQ;
- distributed workers;
- transactional outbox infrastructure;
- Keycloak;
- Kubernetes;
- Redis;
- distributed caching;
- read replicas;
- cross-workbench live dependencies;
- raw user-authored SQL as the Query product model.

Future requirements may add infrastructure when justified.

---

# Part XI — Architecture Decisions and Invariants

## 34. Architecture Decision Summary

Workbench v1 uses:

- MōBrowser 2.x as preferred desktop runtime;
- Electron as primary fallback;
- React + TypeScript for UI;
- application-owned Workbench Core;
- SQLite as primary structured persistence and query engine;
- `.wkbn` as the live SQLite-backed native artifact;
- hybrid relational persistence;
- application-owned typed expression semantics;
- application-owned typed Query Plans;
- hybrid SQLite/application query execution;
- source-owned dependencies with rebuildable dependency indexes;
- explicit derived-state invalidation;
- semantic domain-operation history;
- single authoritative mutation stream per open workbench;
- concurrent bounded reads where appropriate;
- streaming staged imports;
- streaming exports;
- SQLite-backed application-owned search;
- application-owned Grid semantics with an optional third-party Grid implementation;
- background execution as an execution mechanism rather than a separate authority.

---

## 35. Architecture Invariants

The following must remain true unless this architecture is intentionally revised:

1. The v1 product specification is authoritative for product semantics.
2. Core Workbench functionality requires no network or cloud service.
3. React does not own authoritative Workbench state.
4. React does not directly access SQLite.
5. Persistent mutations pass through Workbench Core.
6. Successful persistent domain operations are durably saved as part of completion.
7. Stable UUIDs, not names or positions, define persistent identity.
8. Live dependencies remain within a workbench in v1.
9. SQLite is an implementation mechanism and does not define Workbench semantics.
10. The internal `.wkbn` SQLite schema is private.
11. `.wkbn` is the live workbench artifact, not an export package.
12. Workbench uses bounded data retrieval for large datasets.
13. The UI never requires entire large Tables to be materialized in memory.
14. Query Plans, not raw SQL, define Workbench Queries.
15. Application-side query execution must not silently load unbounded datasets.
16. Formula and Summary definitions are authoritative; calculated materializations are derived.
17. Dependency indexes are derived and rebuildable.
18. Known-stale derived values must not silently be presented as current.
19. Background execution is not an independent mutation authority.
20. Each open workbench has one authoritative mutation stream.
21. History represents semantic domain operations rather than raw persistence changes.
22. Undo and redo execute through Workbench Core.
23. Rebuildable derived state should not be treated as authoritative recovery state.
24. Import and export operate incrementally for large datasets.
25. External formats do not define Workbench data semantics.
26. Grid libraries may implement rendering and interaction but do not own Workbench domain semantics.
27. Accessibility is an architectural requirement.
28. Runtime-specific APIs remain isolated from the Workbench domain where practical.
29. Physical process topology is an implementation decision, not a domain boundary.
30. Infrastructure is added only when it solves a demonstrated problem.

---

## 36. Remaining Implementation Contracts

The architecture intentionally leaves the following implementation details unresolved:

- exact SQLite schema;
- scalar Field physical encodings;
- UUID physical encoding;
- SQLite journaling and connection configuration;
- File Store chunking strategy;
- exact history physical representation;
- history retention thresholds;
- formula materialization heuristics;
- dependency-generation representation;
- query-planner optimization rules;
- search tokenizer and ranking implementation;
- Grid library and renderer;
- MōBrowser background execution topology;
- process/thread/worker placement;
- maintenance thresholds;
- compaction strategy;
- detailed migration algorithms;
- exact large-import batch sizes.

These decisions should be made through implementation design and validation without changing the architectural invariants above.

---

## 37. Pre-Implementation Validation

Before substantial product implementation, Workbench should validate the assumptions identified in `ArchitectureValidationPlan.md`.

A failed validation does not automatically invalidate the architecture.

It should first determine whether:

1. implementation strategy should change;
2. a dependency should change;
3. a runtime-specific capability should change; or
4. an architectural assumption genuinely requires revision.

---