# Workbench — System Design

**Status:** Consolidated V1 architecture baseline (2026-10-12)  
**Companion:** `WorkbenchV1Specs.md`  
**Architecture:** Local-first modular desktop monolith

## About this document

`WorkbenchV1Specs.md` defines product semantics and takes precedence if documents disagree. This design establishes responsibilities, invariants, and execution boundaries. It intentionally does not prescribe every library, physical table, algorithm, dialog, or worker topology. The approved ten-system consolidation supersedes contradictory older design statements, particularly the Visual-only Query model, Grid-only Views, non-blocking validation, import-to-new-Table-only behavior, separate resource packages, and lack of Automations.

## 1. Architecture foundations

### 1.1 Principles

- Workbench must feel like a spreadsheet while maintaining structured-data semantics.
- Core functions are local-native: no account, network, server, or cloud requirement.
- Workbench Core owns domain meaning, validation, persistence, dependencies, History, and mutation authority.
- UI libraries, desktop runtime, and SQLite implement product semantics; they do not define them.
- Domain operations are coherent, durable, auditable units of work.
- Failure is localized; canonical user data is never silently lost or rewritten.
- Bounded reads and background work are required for large datasets.
- Accessibility and recoverability are architecture constraints.
- Add infrastructure when it solves concrete implementation needs, not speculative distributed-system problems.

### 1.2 Responsibility diagram

```text
Operating System / Filesystem
          |
MōBrowser 2.x native shell (Electron fallback)
          |
React + TypeScript UI
  - desktop shell / command registry / editor groups
  - Explorer, Search, Quick Open, Right Panel
  - Grid and other View adapters / Dashboard widgets
  - bounded UI state, focus, selection, session layout
          |
Typed Workbench Core API
          |
Workbench Core — single mutation authority per .wkbn
  - resource & schema services
  - domain operations / validation / transactions
  - relationships / expressions / Query definitions
  - dependency graph / invalidation / freshness
  - History / Undo / Trash / Notes / Automations
  - import/export / portability / file services
  - operation coordinator / search
          |
Data & Query Layer
  - private SQLite schema / typed repositories
  - SQLite read-only query execution
  - indexes, staging, bounded result streams
  - logical File Store for managed bytes
          |
Live versioned SQLite .wkbn

Background workers <--> Core coordinator (never independent writers)
```

The diagram is a responsibility map, not a mandatory process topology. The UI communicates through a typed Core API and never directly opens or mutates SQLite. Workbench's domain Core should remain independent of the desktop runtime wherever practical.

## 2. Desktop runtime, UI, and state ownership

### 2.1 Runtime and windows

Target MōBrowser 2.x. Electron is the primary fallback if a concrete runtime limitation blocks V1; lack of advance validation is not itself a blocker. Encapsulate filesystem, window, clipboard, dialog, file-association, notification, and background APIs behind runtime adapters. Multiple windows may open different Workbenches. Opening an already writable-open file must reuse/activate its existing authority; do not create a second independent writer.

### 2.2 React shell

Provide title/menu area, configurable Command Bar, reorderable Activity Bar, Explorer/Left Panel, two-group tabbed Work Area, contextual Right Panel, Bottom Panel (Problems/Operations/History), and Status Bar. Persist layout presets, panel geometry/visibility, tabs, preview/pinned status, window sessions, and relevant editor-local state. Support customizable shortcuts, Command Palette, context menus, notifications/preferences, and a common command registry.

Explorer is a hierarchy-first navigation surface, not a search or filtering interface. It supports resource browsing, selection, context actions, and hierarchy-aware sorting through its overflow menu. The dedicated Left Search panel owns persistent resource/content search and filtering, while Command Center/Quick Open owns rapid navigation. These surfaces share stable resource identities and navigation commands.

### 2.3 UI versus domain state

UI owns ephemeral focus, selection, scrolling, hover, panel geometry, preview tabs, and bounded presentation caches. Core owns resource identity, schema, Record values, Views' saved definitions, Notes, dependencies, Query definitions, Automation definitions, History, and durable domain state. Session/layout state may persist but must not be confused with canonical resource state. UI components may optimistically display pending edits, but Core acceptance determines the committed result.

### 2.4 Grid and other View adapters

Workbench owns keyboard navigation, logical selection, edit semantics, clipboard behavior, Field/Record identity, and dispatch to Core. A grid library may provide rendering and low-level interaction behind a Workbench-owned adapter. DOM, Canvas, or hybrid implementations remain possible. Virtualization must preserve accessible row/column context. Other native Views—Kanban, Calendar, Gallery, Pivot—consume the same bounded Core data interfaces and editing operations. Field-based grouping and manual ordering are View configuration and/or canonical Field edits, never separate Section membership.

### 2.5 Accessibility

Target WCAG 2.2 AA where applicable. Support keyboard-only navigation, accessible menus/dialogs, screen-reader labels/context, meaningful focus restoration, contrast, and accessibility with virtualized data. Evaluate libraries against these needs rather than delegating product behavior to them.

## 3. Core resource model and identity

### 3.1 Resource graph

```text
Workbench (.wkbn)
├── Collection (non-nestable, organizational)
│   ├── Table ── Field / Record / View
│   ├── Query
│   ├── Dashboard ── Widget
│   └── Automation
├── Table ── Field / Record / View
├── Query
├── Dashboard ── Widget
├── Automation
├── File (managed or external reference; root only)
└── contextual Note (associated with supported target UUID)
```

Tables, Queries, Dashboards, and Automations may exist at root or in a Collection. Collections are **organizational**, not owners of contained resources. Deleting a Collection moves the Collection and all currently contained resources to Trash as one recoverable cascading operation, preserving their hierarchy and UUIDs for restoration, subject to Core validation. Table ownership is genuine and deleting a Table includes owned Fields, Records, and Views. Dashboard-owned Widgets and target-owned Notes follow applicable lifecycle semantics. Individual Cells do not own Notes.

### 3.2 Stable identity

All persistent identity-bearing objects use UUIDs. Resource paths and names are presentation/navigation attributes. Renames, moves, reordering, and changes in display labels preserve identity. Duplication and import create new identities and remap internal references. Source definitions own canonical dependencies; reverse indexes are derived and rebuildable.

### 3.3 Record values and display labels

Logical data is `(recordId, fieldId) -> value`. Physical representation can be hybrid relational, but logical semantics do not depend on SQLite table/column names. Record display labels are configurable and may be composite; references always use UUIDs. Labels need not be unique.

### 3.4 Notes

Store Notes as UUID-identified supporting objects with `target_type` and `target_uuid` (or equivalent typed identity) and content/metadata. Manage through the Right Panel. Notes on Fields and Records are supported; Cell Notes are not. Notes are searchable and follow the lifecycle of their target. Markdown Files remain independent File resources.

## 4. Tables, Fields, validation, and relationships

### 4.1 Field system

Use a typed Field registry/model covering text, numeric, temporal, choice, Boolean, relationships, attachment, JSON, derived, and generated Fields. The registry owns parsing, display, editors, validation, conversion, default behavior, and allowed operations. Specialized presentation types may share physical storage types. Preserve fractional Percentage semantics, configured Currency, wall-clock Time versus instant DateTime, Duration, stable Choice option UUIDs, and blank distinct from zero/false.

### 4.2 Stored versus derived values

Stored Fields contain authoritative values. Formula, Lookup, Rollup/Summary definitions are authoritative; their evaluated results are derived, rebuildable, and freshness-tracked. Generated Fields include sequence, UUID, timestamps, and supported dynamic defaults. Sequence values do not depend on row position. Explicit Convert to Values is a domain operation where supported.

### 4.3 Relationships

Single/multi Linked Record Fields reference one target Table and target Record UUIDs. Reverse navigation and configurable label/candidate policies are Core semantics. A candidate Query must expose stable source Record identity. Candidate filtering does not retroactively destroy existing references. Deletion policy is per relationship: Preserve unresolved reference, Clear, or Restrict; never automatically cascade through links to target Records. Preserve broken links with actionable Problems where policy permits.

### 4.4 Severity-based validation

Validation returns structured findings with Error/Warning severity, target identities, rule identifiers, and actionable messages. Blocking Errors prevent **new** commits; Warnings permit commits and appear in Problems. Structural constraints (UUID integrity, type/storage integrity, foreign-key invariants as modeled) cannot be downgraded. Invalid interactive values remain draft UI state. Existing nonconforming data following schema changes or migration is retained and flagged rather than silently discarded. Schema operations must preview conversions, dependencies, and possible conflicts.

### 4.5 Domain mutation pipeline

```text
User / Import / Automation intent
        ↓
Typed domain operation
        ↓
Prepare and validate proposed final state
        ↓
Analyze dependencies and relationship policies
        ↓
Core-controlled SQLite transaction
   ├── authoritative mutation
   ├── semantic History / Undo metadata
   ├── Notes/ownership lifecycle changes
   └── derived-state invalidation generation
        ↓
Durable commit
        ↓
Publish canonical change + stale dependent state
        ↓
Asynchronous dependency work and UI updates
```

History and authoritative changes commit together. No background worker, SQL Query, Grid adapter, or Automation may bypass this pipeline.

## 5. Views and Dashboards

### 5.1 Saved View definitions

Views belong to Tables and save presentation configuration only. Each Table has a required default View. Persist Grid/Kanban/Calendar/Gallery/Pivot definitions with appropriate filters, sorting, Field configuration, grouping, layout, conditional formatting, and renderer configuration. Opening a Table resolves last-used View then default. View-local selection/scrolling is session state, not part of the saved definition unless explicitly designed as such.

### 5.2 Grouping

Grouping is derived from Field values, supports multiple levels and summaries, and may expose manual order within groups. Dragging a Record between groups is a Field mutation validated through Core. There is **no** separate Section entity, manual Section membership table, or independent Section nesting. Group-creation UI may create/configure a choice Field and must make canonical effects clear.

### 5.3 Formatting

Formatting/renderer definitions are declarative and non-executable, with reusable presets, conditional rules, and View overrides. Their evaluation is bounded, accessible, and tied to dependency/freshness semantics. Formatting cannot alter canonical values.

### 5.4 Dashboard architecture

Dashboards own widget definitions in a responsive 12-column layout. Widgets read Tables, Views, or Queries through bounded data APIs and support charts, KPIs, tables, text, shared/local filters, cross-filtering, and drill-through. Widget errors and broken dependencies remain visible in Problems. Source editing, when available, dispatches normal Core operations rather than mutating query results.

## 6. Queries and typed expressions

### 6.1 One Query resource, two definition modes

A Query has `mode = Visual | SQL` (or equivalent), a stable UUID, dependencies, result schema, and status. **Visual mode** stores an application-owned typed Query Plan; it may display generated SQL read-only. **SQL mode** stores user-authored controlled SQLite SELECT text. A user may Duplicate as SQL Query, creating a separate resource; two-way round-trip conversion is not promised.

### 6.2 Visual planner

Visual Queries support sources, joins, projections, filters, computed expressions, grouping, aggregation, sorting, and limits. The planner pushes semantically compatible operations into SQLite; application-owned evaluation may supplement this only with bounded/streamed execution. Do not silently materialize huge datasets in application memory.

### 6.3 SQL execution boundary

SQL mode must be restricted to read-only statements and approved logical dataset interfaces. Enforce SQLite read-only/authorizer protections (or equivalent defense in depth), controlled dataset exposure, cancellation, time/memory/result budgets, and prevention of access to private internal tables or dangerous functions. SQL is not an escape hatch to bypass Core. Supported CTEs, joins, expressions, and subqueries use native SQLite semantics. Query-to-Query references resolve through a safe, cycle-checked dependency model.

### 6.4 Typed expression engine

One application-owned typed expression model serves Formulas, Visual Query expressions, validation, conditional formatting, and Automation conditions where appropriate. Bind Field references to UUIDs after parsing readable notation such as `[Quantity]`. Define type coercion, null/blank behavior, numeric precision, errors, and context-allowed functions explicitly. SQL mode intentionally uses SQLite semantics; translation must preserve Workbench semantics where claimed.

### 6.5 Query results

Results are read-only typed virtual datasets. Provide bounded windows/streams, pagination, error and stale/fresh states, dependency metadata, and source Record identity when meaningful. Query results with stable Record identity may supply Linked Record candidate sets. Aggregated result rows cannot masquerade as source Records.

## 7. Persistence and native artifact

### 7.1 `.wkbn` contract

`.wkbn` is the **authoritative live SQLite database**, not a ZIP, sidecar bundle, or export archive. The schema is private and versioned. SQLite stores structured resources, history, configuration, metadata, and managed File content. One Core mutation authority exists per open workbench.

### 7.2 Hybrid relational model

Use table-oriented physical storage for ordinary scalar Fields where useful for SQLite retrieval and Query execution, with specialized storage for multi-value links, metadata, Notes, Files, history, and derived indexes. Maintain a mapping between stable domain UUIDs and private physical structures. Renaming a user Field must not invalidate its identity. Physical schema design remains an implementation choice.

### 7.3 File Store

Expose separate logical Structured Repository and File Store abstractions backed by the same `.wkbn`. Managed File bytes are embedded; external Files retain references and availability status. Replacing File content preserves File UUID. Chunking/streaming strategy is chosen for scale and safety.

### 7.4 Transactions and concurrent reads

Serialize persistent mutations through Core. Allow bounded concurrent reads through suitable SQLite read connections where appropriate. Long work should stage/prepare outside short authoritative publication transactions. SQLite journaling, busy handling, checkpointing, and process/thread placement are implementation details subject to durability guarantees.

### 7.5 Encryption

Optional encryption protects the `.wkbn` at rest, including managed File content. Choose a supported encrypted SQLite mechanism with clear key handling, compatibility, and migration behavior. Do not promise protection for external File references, temporary external exports, or backups unless they are separately protected. Make encryption state explicit in copy/backup workflows.

## 8. Import, export, clipboard, and portability

### 8.1 Shared operation pipeline

Parse → map → stage → preview → validate final state → commit. Simple edits may omit unnecessary UI steps. Large operations stream/stage bounded data. Import adapters support CSV/TSV/XLSX/JSON, type inference, new Table creation, append, and upsert. Exports use bounded Core streams and support selected/visible versus all-data semantics.

### 8.2 Explicit partial acceptance

For imports, validate all proposed rows, classify Errors and Warnings, and allow the user to select a valid subset for **one atomic commit**. Rejected rows remain in staging for correction/export. Never silently skip rows. For upsert, compute final-state validation against affected existing Records and cross-row uniqueness/relationship rules. An accepted subset must be internally consistent; conflicts are resolved before publication.

### 8.3 Clipboard, bulk edits, and transforms

Internal clipboard preserves typed values and supported identity/reference semantics; external text/HTML representations support spreadsheet interoperability. Bulk edits, Find/Replace, and expression transforms preview effects and use coherent Core transactions and History. Protect calculated/system Fields.

### 8.4 Native portability

Save a Copy and resource-transfer packages both use **valid `.wkbn` SQLite databases**. A transfer file is a valid Workbench containing the selected resources and included dependencies. Import generates/remaps UUIDs and reports unresolved dependencies. Preview external Files with per-item embed decisions and Include All. Unembedded external references remain explicit. Template instantiation creates fresh identity and follows Automation trust rules.

## 9. Dependency graph, execution, and freshness

### 9.1 Source-owned dependency graph

Canonical definitions store their own dependency references. Build a reverse-dependency index for invalidation, impact analysis, and navigation; it is derived and rebuildable. Detect cycles in Formulas, Queries, and supported Automation dependencies before execution.

### 9.2 Transactional writes and asynchronous derived results

A successful canonical write commits promptly with dependency invalidation. Formula/Lookup/Rollup/Query/Dashboard results may update asynchronously. Each derived result has an explicit generation or equivalent freshness token and status: stale, computing, fresh, or error. A result computed against an older generation must never replace a newer one. A consumer requiring freshness waits for the correct generation or reports a bounded failure.

### 9.3 Operation coordinator

Coordinate interactive reads, serialized writes, calculation, SQL, imports, exports, indexing, backups, maintenance, and Automations. Support priority, bounded concurrency, progress, cancellation, and resource budgets. Workers can read/prepare/stage; only Core publishes authoritative mutations. Do not prescribe a custom scheduler or worker topology without an implementation need.

### 9.4 Indexes and caches

Maintain identity/relationship indexes, search indexes, and workload-appropriate SQLite indexes. Materialized derived state is rebuildable, versioned, and invalidated. Caches do not become a competing authority. Sorting, filtering, grouping, and aggregation should execute near data where appropriate.

### 9.5 Scale expectations

Preserve 1M routine, 10M primary V1 target, 50M stress target, and 100M+ best-effort Record expectations. These are storage/processing expectations, not a requirement to render all rows. Bounded reads and streaming are non-negotiable; exact performance targets should be measured during implementation rather than requiring a separate pre-development validation phase.

## 10. Automations

### 10.1 Declarative resource model

Automations are UUID-backed root/Collection resources. A visual definition contains supported triggers, conditions, actions, enabled state, schedule/missed-run policy, dependencies, and run metadata. Actions dispatch approved Core operations; arbitrary scripts/native code are not supported.

### 10.2 Execution semantics

Automation runs use the same validation, relationship integrity, write serialization, and History boundaries as interactive operations. Define deterministic ordering, conflict handling, recursion/retrigger limits, and safe cancellation. Runs that consume derived results wait for the required fresh generation or fail visibly.

### 10.3 Trust-aware activation

Enabled state in a transferred `.wkbn` is not sufficient authorization on a new local machine/context. Locally authorized Automations may resume; imported or otherwise untrusted Automations remain disabled until reviewed and enabled. The local authorization record is separate from blindly copied artifact content. Destructive or environment-dependent actions require explicit safeguards.

### 10.4 Desktop-local schedules

Scheduled Automations run only while the relevant Workbench Core is active. Closing the Workbench does not install a hidden background service. Persist schedule and last-run state, expose missed runs, and apply the configured missed-run policy when reopened. No cloud/always-on guarantee.

### 10.5 Run visibility

Provide test/manual run, enable/disable, logs, timestamps, affected resources, errors, and supported Undo/History. Integrate with Operations, Problems, Status Bar, and notifications.

## 11. History, Trash, backups, and recovery

### 11.1 Semantic History and Undo

Record domain-level changes rather than raw SQL mutations. Commit History atomically with canonical mutations. Undo/Redo are Core domain operations subject to current dependencies and constraints, with explicit conflict reporting. Retention/compaction are implementation choices.

### 11.2 Trash and ownership

Trash preserves recoverable deleted resources and genuinely owned descendants. A Table deletion includes its Fields/Records/Views; relationships use Preserve/Clear/Restrict and do not cascade to linked target Records. 

The Core should determine the affected resource set, validate dependencies, and move the Collection and contained resources to Trash transactionally. Restoration should restore the group consistently rather than creating partially restored Collections.

No separate Archive state; use Collections and Explorer visibility for inactive resources.

### 11.3 Backups and recovery

Use SQLite-consistent backup mechanisms, including when a Workbench is open. Support checkpoints, retention, backup-before-migration, recovery from interruption, and integrity checks. Backups, Trash, Undo, and History serve different purposes. Derived caches/indexes can be rebuilt; do not confuse them with authoritative recovery state.

### 11.4 Versioning and migration

Version `.wkbn` schema and artifact contracts. Opening older supported versions runs safe migrations with backups and explicit reporting. Unsupported newer versions must not be silently rewritten or downgraded. Preserve identity and recoverable data. Maintain compatibility with optional encryption and transfer `.wkbn` files.

## 12. Search, Problems, and diagnostics

Search is distinct from structured Queries and supports resources, Records, and contextual Notes through a Core Search Service. SQLite FTS or equivalent indexing may be used; indexes are rebuildable and results bounded. Problems are persistent/actionable Error or Warning findings for invalid values, unresolved references, expression failures, Query/Widget issues, missing Files, migrations, and Automation errors. Status Bar and Bottom Panel show counts and navigable targets. Operations report queued/running/completed/failed/canceled state. Diagnostics must not overwhelm normal workflows.

## 13. Implementation contracts and boundaries

The following remain engineering choices to resolve during implementation, not blockers to starting development:

- Exact private SQLite schema, hybrid storage mapping, UUID encoding, and indexes.
- SQLite connection/journaling/checkpoint configuration and locking behavior.
- MōBrowser native module packaging and runtime API integration; Electron fallback only on a concrete blocker.
- Grid library/adapter, rendering strategy, virtualization, and accessible interaction mechanics.
- UI state library, routing, editor component choices, and command registry implementation.
- Expression parser/evaluator, Query Plan lowering, SQL dataset exposure, and read-only SQL authorizer design.
- File Store binary streaming/chunking, encryption provider, and key lifecycle.
- History physical representation, backup retention, migration steps, and cleanup thresholds.
- Dependency generation representation, cache policies, and background worker topology.
- Import/export adapters, batch sizing, staging, and upsert conflict-resolution implementation.
- Automation trigger queue, missed-run policy representation, and local trust store.

These choices should be made pragmatically in code and supporting design notes, not through an obligatory pre-implementation validation phase.

## 14. Architecture invariants

1. V1 product semantics in `WorkbenchV1Specs.md` are authoritative.
2. Offline local operation is complete without accounts or servers.
3. `.wkbn` is the live SQLite artifact, including valid resource-transfer copies.
4. Each open Workbench has one Core mutation authority.
5. React and runtime adapters do not directly mutate SQLite.
6. UUID identity survives rename/move; duplication/import remaps identities.
7. Collections organize, Tables own; Collection deletion cascades through contained resources, while genuinely owned descendants follow their ownership lifecycle.
8. Notes attach to objects, not individual Cells.
9. Views present canonical Table data; no manual Sections domain model.
10. Visual and SQL Queries are distinct definitions of one Query resource type.
11. SQL is read-only and cannot bypass Core or access private storage arbitrarily.
12. Errors block new commits; Warnings do not; pre-existing nonconforming data remains recoverable.
13. Import partial acceptance is explicit and atomically commits a valid subset.
14. Canonical writes commit atomically; derived updates may be asynchronous but never silently stale.
15. All persistent changes, including Automation actions, pass through Core and meaningful History.
16. Imported/untrusted Automations are disabled pending local review; schedules run only while Core is active.
17. Native transfer uses `.wkbn`, not a separate resource-package format.
18. History, Trash, and Backups are distinct recovery mechanisms; no separate Archive lifecycle.
19. UI reads are bounded and accessible at large scale.
20. MōBrowser is preferred, Electron is fallback, and implementation begins without mandatory pre-development validation exercises.

## 15. Practical implementation order

1. Create the React/TypeScript desktop shell and typed runtime/Core boundary.
2. Implement `.wkbn` create/open/close, SQLite authority, and basic durable domain operations.
3. Implement UUID resource graph, Collections, Tables, Fields, Records, Views, and contextual Notes.
4. Connect Explorer, tabs, default View resolution, and a bounded editable Grid to real Core operations.
5. Expand Field types, relationships, validation, History, and Problems.
6. Add remaining View types, Queries (Visual/SQL), typed expressions, and Dashboards.
7. Add import/export, clipboard, portability, Files, templates, and recovery/encryption.
8. Add coordinated background execution, freshness, Automations, and final accessibility/performance work as dependencies demand.

This sequence is an implementation path, not a V1 feature reduction or a requirement to finish every subsystem before integrating the next.
