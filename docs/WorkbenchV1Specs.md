# Workbench Version 1 Specifications

**Status:** V1 product specification — revised architecture baseline  
**Companion:** `docs/SystemDesign.md`  

> Workbench should feel like a spreadsheet and behave like structured data underneath.

## 1. Product Definition

Workbench is a local-first desktop application for building, organizing, analyzing, and visualizing structured data. The grid is a primary interaction surface, not the underlying data model. A **workbench** is a portable, self-contained `.wkbn` artifact; **Workbench** names the application. V1 operates offline and does not require accounts, servers, or cloud synchronization. A `.wkbn` file is the live versioned artifact, physically a SQLite database with a private Workbench-owned schema, not an export archive or container around another working database.

## 2. Core Object Model

```text
Workbench
├── Collection (non-nestable)
│   ├── Table
│   ├── Query
│   └── Dashboard
├── Table
│   ├── Field
│   ├── Record
│   └── View
├── Query
├── Dashboard
│   └── Widget
└── File
```

Tables, Queries, and Dashboards can be at workbench root or inside a Collection. Collections cannot nest. Files are workbench-level resources. A Table owns Fields, Records, and Views. A View is a saved presentation of one Table; a Query is a live read-only derived dataset; a Dashboard contains Widgets. A value is logically `(recordId, fieldId) → value`; a grid cell is its presentation, not a separate authoritative object.

## 3. Identity and Dependencies

Persisted resources, Fields, Records, options, and other identity-bearing objects use stable internal UUIDs independent of names, locations, and positions. Renames, moves, sorting, and filtering do not alter identity. Definitions own their dependency references; a rebuildable reverse-dependency index accelerates impact analysis. Broken dependencies are preserved where practical and surfaced in Problems, not silently erased. Destructive operations warn about affected dependents.

## 4. Workbench Boundary

Live references, Formulas, Summaries, Queries, and Dashboard dependencies are restricted to one workbench. Cross-workbench live links are excluded in V1. Import/export/native sharing can move snapshots between workbenches, with local identity remapping as required.

## 5. Desktop and Window Model

Each desktop window displays one workbench or Home. Multiple windows can show different workbenches; one workbench must not be independently open for writing in two windows. Opening an already-open artifact activates its existing window. Opening another workbench offers This Window or New Window. Direct application launch attempts to reopen the most recent workbench, otherwise shows Home. Closing a window does not require a traditional save prompt.

## 6. Workbench Creation and Saving

New workbenches require a name and file location before creation; there is no normal untitled unsaved state. Autosave is the persistence model: successful persistent domain operations durably commit their authoritative changes and required history together. React does not own an unsaved authoritative duplicate. `Ctrl/Cmd+S` may offer an explicit checkpoint/flush action but is not required to preserve ordinary edits.

## 7. Application Shell

The React/TypeScript shell includes title/menu area, command interface, Activity Bar, Left Panel, tabbed Work Area, Bottom Panel, Right Panel, and Status Bar. Explorer is the primary Left Panel surface; Problems appears in Bottom Panel; Status Bar communicates persistent state and background progress. UI layout and transient interaction belong to React, while saved resource configuration belongs to Core.

## 8. Tabs and Resource Opening

Tables open through Views, not generic Table tabs. Each Table has a default View; opening a Table selects its last-used View when available, otherwise default. Views, Queries, Dashboards, and Files can open in tabs. Single-click preview may reuse one preview tab; editing, double-clicking, or pinning promotes it. Existing normal tabs are activated rather than duplicated. Closing a tab does not delete or unsave the resource. Tab order and active context may restore on reopening.

## 9. Explorer

Explorer shows Collections first, followed by root Tables/Queries/Dashboards grouped by type and alphabetized. Collection contents are type-grouped and alphabetized. Tables expand to show Views, with default View first. Collections expand/collapse and do not open as content tabs. Single-click openable resources previews; double-click opens normally. Files appear in a separate flat Files section, filterable by name/type/date. Explorer location is not identity.

## 10. Naming

Names are human-readable and case-insensitively unique among same-type siblings. Different resource types may share a name under the same parent. Collections are unique at workbench root; Fields are unique within a Table; Views within a Table; Choice options within their Field. Rename preserves stable identity and dependent references. Moving resources checks destination name conflicts.

## 11. Tables and Grid Philosophy

The Grid is finite and structured: rows are Records, columns are Fields, and values are Record–Field intersections. Row numbers are presentation positions, never identifiers; Field names replace spreadsheet letters as domain references. Empty Tables start with no Fields or Records. Virtual blank rows can fill viewport space but are not stored until edited. There are no virtual blank Fields. Familiar keyboard navigation, selection, editing, paste, and bulk operations should work where compatible with structured semantics.

## 12. Table and Field Creation

A Table requires a name and can be created as Empty Table, Configure Table, or Import. Empty Tables start with zero Fields/Records. Configure Table defines initial schema. Import creates a new Table after preview and configuration. Grid Add Field creates a named Field, defaulting to Text. Pasting into an empty Table can propose Fields/types, with user review before committing schema.

## 13. V1 Field Types

Supported Field types:

| Family        | Types                                |
| ------------- | ------------------------------------ |
| Text          | Text, Long Text, Email, Phone, URL   |
| Numeric       | Number, Currency, Percentage, Rating |
| Boolean       | Boolean, Checkbox                    |
| Temporal      | Date, Date & Time, Time, Duration    |
| Choice        | Choice, Multi-Choice                 |
| Relationships | Reference, Multi-Reference, Summary  |
| Derived       | Formula                              |
| Content       | Attachment, JSON                     |
| System-owned  | Generated                            |

Currency is Field-level single-currency configuration; Percentage uses fractional underlying semantics (12.5% = 0.125); blank differs from false or zero. Date & Time denotes an instant, Time a wall-clock time, Duration elapsed time. Choice options have stable IDs and unique names. JSON is validated structured content but nested keys are not Fields. Generated modes: Sequence, UUID, Created Time, Modified Time. Geographic Field type is excluded in V1. Types own validation/editor/formatting semantics rather than scattering behavior through UI.

## 14. References and Relationships

A Reference stores zero or one target Record in one configured Table; Multi-Reference stores zero or more target Records from one Table. Relationships use target IDs, not display labels. Each Reference Field chooses a display Field, which may be suggested automatically and can differ among References to the same Table. Reference traversal can expose target Fields. Derived referenced values are read-only at the referencing location; edit the source Record. Deleting a target preserves an identifiable broken relationship and surfaces Problems. Polymorphic multi-Table References are excluded in V1.

## 15. Summary Fields

A Summary is a read-only aggregate over Records reached through a multi-record relationship. It specifies the relationship, source Field/Records, and type-compatible aggregation such as Count, Sum, Average, Minimum, or Maximum. Summary definitions are authoritative; results are derived, optionally cached/materialized/indexed, and rebuildable. Source edits invalidate affected results. Known-stale results cannot silently appear current. Explicit Convert to values may snapshot current results into stored values.

## 16. Formula Fields

Formula expressions begin with `=` and reference Fields by human-readable bracket notation, e.g. `=[Quantity] * [Unit Price]`; names bind internally to stable Field IDs. Reference traversal may use `=[Customer].[Discount Rate] * [Subtotal]`. Formulas evaluate current-Record values and relationships, not spreadsheet coordinates or arbitrary cell ranges. Set aggregates belong in Summaries/Queries. The application-owned typed expression engine defines semantics; SQLite may execute compatible translated expressions without redefining them. Cycles are invalid. Definitions are authoritative; results are read-only, rebuildable, invalidated on change, and may be converted explicitly to stored values. Errors are surfaced in Problems.

## 17. Generated Fields

Generated modes are Sequence, UUID, Created Time, and Modified Time. Values are system-owned and normally read-only. Sequence can be numeric or formatted and does not follow row position. A user-facing UUID Field is distinct from the internal Record ID. Duplicating a Record generates new appropriate values. Convert to values may remove generation behavior. Created By and Modified By are excluded from V1.

## 18. Attachments and Files

Files are stable-identity workbench-level resources embedded in `.wkbn`. Attachment values reference zero or more Files and do not duplicate File bytes. Plain-text viewing/editing may be supported in-app; other types can open externally. File metadata and binary content use a logical File Store abstraction physically backed by SQLite inside `.wkbn`. Chunking/streaming representation is an implementation decision. Deletion of referenced Files is dependency-aware. Embedded content is data, never implicitly executable code.

## 19. Required Fields and Validation

Stored Fields may be Required. Missing/invalid values are preserved and surfaced as persistent Problems where safe, rather than silently replaced or routinely blocking incremental data entry. Semantic validators apply to Email, URL, numeric, Choice, Reference, JSON, and other types. Blank is distinct from explicit empty/zero/false values. Fixing a condition automatically resolves its Problem.

## 20. Field Type Conversion and Defaults

Type conversion attempts to preserve values. Convertible values are converted; incompatible originals remain identifiable and correctable as Problems, with impact preview for broad changes. Compatible type transitions may be cheaper than semantic conversions. Derived/system Fields may offer explicit Convert to values. Stored Fields may have defaults applied to newly created Records; changing defaults does not rewrite existing Records. Derived Fields do not use ordinary stored defaults.

## 21. Record Operations

Records have stable IDs and can be created, edited, duplicated, deleted, and bulk-modified. Duplicating copies stored editable values and existing Reference/File relationships without duplicating targets; system values regenerate and derived values recalculate. Deletion is recoverable through Table history rather than resource Trash. Broken inbound References remain identifiable. Bulk operations use coherent domain-operation/transaction semantics; large work may stage or batch internally before atomic publication.

## 22. Views

A View is a saved presentation of one Table and does not duplicate Table data. It can define Field visibility/order/widths, frozen Fields, filters, sorting, grouping, density, row height, and conditional formatting. View configuration autosaves. V1 ships Grid View; other View Types may be added later. Multi-level grouping and type-appropriate group aggregates are presentation-only, distinct from Summary Fields. Each Table has a default View; last-used View may be reopened.

## 23. Queries

Queries are saved live read-only derived datasets from Tables and/or Queries. They can project, filter, join, calculate, group, aggregate, sort, and combine sources through a visual builder. Definitions use an application-owned typed Query Plan, not raw user SQL. Query dependency cycles are invalid. Existing References can inform join suggestions; ad-hoc compatible joins may be supported. Execution pushes semantically compatible work into SQLite and uses bounded application-owned evaluation for specialized behavior. Unbounded memory fallback is prohibited. Source changes propagate; navigation to editable source Records is preferred over editing Query results. SQL authoring is outside V1.

## 24. Dashboards

Dashboards contain Widgets that present live data from Tables/Queries without owning source Records. Core Widget classes include charts, KPI/value cards, tabular presentations, and text/labels. Widgets can configure placement, sizing, labels, and visualization. Broken sources remain identifiable and surface Problems. Dashboard-driven source editing and advanced global interactive filtering are outside V1.

## 25. Problems

Problems are persistent actionable issues, not transient notifications. V1 severities are Error and Warning. Examples include invalid values, Required violations, conversion failures, broken References, Formula errors, and invalid Query/Widget dependencies. Status Bar shows counts; Bottom Panel lists Problems and navigates to affected resources/Records, opening previews as needed. Problems derive from authoritative state and disappear automatically when fixed. Results are bounded for large workbenches.

## 26. Import

Import converts external structured data into a **new Table** in V1; it does not merge/synchronize with existing Tables. Uploading a File is distinct from importing data. Import offers source preview, parsing options, Field names/types, and type-inference review. Format adapters parse/validate incrementally into non-authoritative staging. Publication is one coherent domain operation. Cancellation/failure before publication leaves authoritative state unchanged. Progress, responsiveness, and bounded memory are required for large imports.

## 27. Export

Export creates interoperable external representations, distinct from native sharing. View export defaults to visible filtered/ordered Fields and Records, with Include all data to export the underlying Table. Query results can export where supported. Derived values may export as current values. Exporters consume bounded streams through Workbench APIs, not the private SQLite schema; large exports support progress and safe cancellation.

## 28. Native Sharing

Native sharing produces a separate snapshot **resource package** (format/extension TBD), not a `.wkbn` working artifact. It includes selected resources and recursively required dependencies, with contents inspectable before import/share. Import creates local duplicate resources with new UUIDs, remaps internal links, resolves naming conflicts without overwriting, and does not create live links to source workbenches. The resource-package format remains an implementation/product-detail question independent of `.wkbn`'s locked SQLite format.

## 29. Trash and Deletion

Resource-level deletion uses Trash for recoverable Tables, Queries, Dashboards, Collections, Files, and appropriate owned resources. Deletion warns about dependent resources rather than silently cascading. Restore preserves IDs where practical and resolves unavailable original locations. Permanent deletion is explicit. Record deletion instead uses Table history. Trash and history may have distinct retention policies.

## 30. History and Recovery

History is semantic and domain-operation based, separate from short-term resource-scoped undo/redo. Persistent mutations atomically commit authoritative state, required history metadata, and undo/recovery information. Undo/redo execute through Core, not raw SQL reversal. Physical recovery representation may use inverse data, before-state, snapshots, or specialized restoration. Durable history supports recovering deleted Records and bulk mistakes across restarts. It is not Git-style branching/version control. Retention and compaction are permitted; details remain implementation decisions.

## 31. `.wkbn` Artifact

A `.wkbn` file is the live authoritative, portable, versioned SQLite-backed Workbench artifact. It is **not** a ZIP archive, directory bundle, exported container, or wrapper around a separate working database.

```text
Finance.wkbn
└── SQLite database (private Workbench schema)
    ├── format / domain metadata
    ├── structured Tables / Records / relationships
    ├── Queries / Views / Dashboards
    ├── dependency and derived state
    ├── history / recovery state
    └── embedded File metadata and binary contents
```

Logical Structured Repository and File Store are separate application-owned abstractions, both physically stored inside SQLite. Scalar Fields use hybrid table-oriented relational storage; specialized structures serve multi-values, relationships, metadata, history, and Files. The logical `(recordId, fieldId) → value` model remains independent of schema. Stable IDs, not user-facing names, govern physical identity. SQLite is used for transactional persistence and query execution but does not define product semantics. The schema is private; direct SQL modification is unsupported. Temporary non-authoritative files may exist outside `.wkbn` but must not compromise healthy-artifact portability. Copy/move/backup of a healthy workbench must preserve required state; implementation must account for SQLite journaling/checkpoint safety.

## 32. Inspectability and Safety

Workbench exposes understandable contents and integrity/version information for workbenches and native packages, including resources and embedded File metadata. Users should be able to review shared contents, especially embedded Files, without editing private SQLite internals. Package import validates contents before authoritative publication. Embedded Files do not gain executable privileges by being opened as data. Unsupported/corrupt artifacts are handled conservatively.

## 33. Offline-First

Core operations—including open/create, editing, relationships, Formulas, Summaries, Queries, Dashboards, import/export, embedded Files, History, and Problems—work without a network, account, or remote database. Local `.wkbn` is authoritative. Future online integrations must not turn unrelated local workflows into network-dependent workflows. V1 excludes sync and collaboration.

## 34. Persistence and Crash Safety

Persistent changes pass through one Core mutation authority per open workbench. SQLite transactions provide normal atomicity; committed operations survive and uncommitted operations do not partially publish. Bounded concurrent reads may use appropriate connections. Long work stages/prepares outside short publication transactions when practical. Derived indexes/materializations are rebuildable. Crashes, failed imports, and maintenance interruption must leave authoritative state recoverable. Corruption detection avoids speculative destructive repair. Structured metadata and embedded File writes must remain transactionally consistent.

## 35. Format Versioning and Migration

`.wkbn` has an explicit version and private SQLite schema. Supported older versions migrate deterministically with a recoverable pre-migration state. Failed migration must not knowingly destroy the last usable artifact. Newer unsupported formats must not be modified as if understood. Internal schema can evolve; interoperability occurs through Workbench APIs and supported external/native formats. Native resource packages have their own versioning as appropriate.

## 36. Scale Requirements

|      Records | Engineering expectation |
| -----------: | ----------------------- |
|    1 million | Routine                 |
|   10 million | Primary V1 target       |
|   50 million | Stress target           |
| 100 million+ | Best effort             |

Targets describe storage/processing, not simultaneous rendering. Operations may become slower with scale, but the architecture must avoid artificial frontend or domain-model ceilings. Performance comes from SQLite set operations, indexing, bounded retrieval, streaming, virtualization, and measured derived materialization.

## 37. Bounded Frontend Data

React/Grid requests bounded windows of visible data and relevant metadata. It never owns whole large Tables or unbounded Query/Search results. Filtering, sorting, grouping, aggregation, and search should execute in the data/query layer when appropriate. React communicates through an application-owned typed Core API and never directly accesses SQLite. Presentation caches remain bounded and non-authoritative.

## 38. Long-Running Operations

Imports, exports, queries, recalculation, conversions, bulk edits, indexing, maintenance, integrity checks, and migrations execute without blocking the UI. Background Execution is an execution mechanism, not a second mutation authority. Workers may read/prepare/stage, but authoritative publication passes through Core. Long work offers progress and safe cancellation where practical. Interrupted derived work is marked invalid and rebuildable; stale results cannot silently be presented as current. Exact threads/processes/workers are implementation decisions.

## 39. Desktop Technology

Workbench V1 targets **MōBrowser 2.x**. **Electron** is the primary fallback only if concrete validation finds a material MōBrowser blocker; undocumented behavior is not itself a limitation. Validate native SQLite module packaging, background execution, file associations, multi-window behavior, cross-platform distribution, licensing, debugging, and performance. Runtime-specific APIs remain isolated from Core. Physical execution topology is chosen from measured requirements, not mandated as a separate Core process.

## 40. Frontend Technology

React + TypeScript implement application shell and UI. React owns tabs, panels, focus, selection presentation, scrolling, menus, and bounded displayed data; Workbench Core owns authoritative domain state, domain operations, Query/Formula semantics, history, and persistence. Build/state-management libraries remain implementation choices. Specialized high-performance rendering may coexist with React without making React authoritative for large datasets.

## 41. Grid Architecture

Workbench owns Grid semantics: navigation, selection, editing, clipboard, Field/Record identity, and domain operation dispatch. A third-party high-performance grid library **may** implement rendering/interaction behind a Workbench-owned Grid Adapter. The library must not own authoritative data, Formulas, Queries, dependencies, history, or persistence. DOM, Canvas, hybrid, and specific libraries remain open for evaluation. Bounded data windows, logical scrolling, keyboard access, and screen-reader behavior are mandatory evaluation criteria. Renderer choice must not leak into Core semantics.

## 42. Accessibility

Accessibility is architectural, targeting WCAG 2.2 AA where applicable. Keyboard-only workflows, focus management, meaningful roles/labels, contrast, screen-reader context, accessible dialogs/menus, and Grid navigation are required. Renderer and grid-library selection must validate accessibility rather than defer it. Large-data virtualization must preserve meaningful accessible context.

## 43. Architecture and Ownership

```text
Operating System
      ↓
MōBrowser / Native Shell
      ↓
React UI
      ↓
Workbench Core API
      ↓
Workbench Core
├── Application Services
├── Domain Operations
├── Expression / Dependency / History / Search
      ↓
Data / Query Layer
      ↓
SQLite
      ↓
.wkbn

Background Execution ↔ Core / Data / Query
```

This is a **modular desktop monolith**, not a distributed service architecture. The diagram denotes responsibilities, not mandatory processes. One authoritative mutation stream exists per open workbench. Workbench owns typed expressions, typed Query Plans, domain operations, dependencies, and recovery semantics. SQLite owns transactional local storage and compatible query execution. Third-party libraries supply commodity capabilities without defining product meaning.

## 44. V1 Exclusions

V1 does not include cloud synchronization, real-time collaboration, accounts/permissions, cross-workbench live links, arbitrary SQL authoring, server deployment, general-purpose document editing, advanced Dashboard interactivity, polymorphic multi-Table References, third-party Field plugin marketplace, or spreadsheet-coordinate formulas. These exclusions do not prohibit forward-compatible design where it remains inexpensive and justified.

## 45. Open Implementation Questions

The following remain implementation choices, **not** unresolved decisions about whether `.wkbn` is SQLite-backed:

- exact private SQLite schema and Field storage encodings;
- UUID encoding and schema migration mechanics;
- SQLite journaling, checkpointing, connection topology, and file-copy safety;
- embedded File chunking/streaming representation;
- semantic history physical representation and retention thresholds;
- dependency generation/invalidation representation;
- Formula/Summary materialization heuristics;
- typed Query Plan translation and planner optimizations;
- search indexing/tokenization and ranking;
- Grid library and DOM/Canvas/hybrid renderer;
- MōBrowser native SQLite and background-execution packaging validation;
- worker/thread/process placement;
- maintenance, reclamation, compaction, and integrity thresholds;
- exact native **resource-sharing package** format (distinct from `.wkbn`);
- detailed import/export adapter and batching choices.

Do not reopen the locked `.wkbn` SQLite artifact decision merely because physical details are pending.

## 46. Governing Principles and Acceptance

V1 implementation must preserve these invariants:

1. Product semantics are authoritative over framework conveniences.
2. Local operation is first-class and does not require cloud services.
3. Stable UUIDs define identity; names and positions do not.
4. Domain operations represent meaningful persistent changes.
5. Autosave means successful persistent operations are durable.
6. One Core mutation authority exists per open workbench.
7. React is presentation, not authoritative persistence or query state.
8. `.wkbn` is the live self-contained SQLite-backed native artifact.
9. SQLite schema is private and does not define Workbench semantics.
10. Structured Store and File Store remain logically distinct inside `.wkbn`.
11. Formulas, Summaries, Query Plans, dependencies, and history are application-owned.
12. Derived state is explicitly invalidated and rebuildable; known-stale values are never silently current.
13. Background work cannot bypass authoritative domain transactions.
14. Large datasets are handled through bounded data and streaming, not frontend materialization.
15. Imports stage and publish atomically; exports stream bounded results.
16. Grid libraries may render and interact but do not own Workbench semantics.
17. Accessibility and recoverability are design constraints, not optional polish.
18. MōBrowser is the V1 runtime target; Electron is a concrete-blocker fallback.
19. Runtime-specific APIs stay out of domain Core where practical.
20. Infrastructure is added to solve validated problems, not hypothetical distributed-system needs.

**Implementation should follow `docs/SystemDesign.md`; unresolved technical assumptions should be tested through `docs/ArchitectureValidationPlan.md`.**
