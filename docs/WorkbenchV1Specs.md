# Workbench — Version 1 Product Specifications

**Status:** Consolidated V1 product baseline (ten systems approved, 2026-10-12)  
**Companion:** `SystemDesign.md`  
**Principle:** *Feel like a spreadsheet; behave like structured data underneath.*

## Purpose, scope, and interpretation

Workbench is a local-first desktop application for building, organizing, editing, querying, visualizing, and automating structured data. The grid is an interaction surface; Tables, Fields, Records, and relationships are the underlying domain model. Each workbench is a live, versioned `.wkbn` SQLite database. V1 works offline without accounts, cloud sync, or a server. Live references stay within one workbench.

This document consolidates the approved V1 decisions into ten systems. **Consolidation does not remove previously approved V1 features** except where a later explicit decision replaced an earlier model: no standalone manual Sections, no separate Archive lifecycle, and no separate native resource-package format. Implementation details not specified here may be selected during development, provided product behavior is preserved. The companion system design describes implementation boundaries; this document governs product semantics.

## 1. Desktop & Workspace — seven requirements

### 1A. Configurable desktop shell
The desktop includes title/menu area, optional Command Bar, Activity Bar, Explorer/Left Panel, tabbed Work Area, contextual Right Panel, Bottom Panel, and Status Bar. Panels can be shown/hidden, resized, and persisted. The Work Area has layout priority. The Activity Bar is reorderable; named layout presets are supported. Panels and layouts remain keyboard-accessible.

### 1B. Resource-oriented Work Area
Tables open through Views. Views, Queries, Dashboards, Files, and tools occupy preview, permanent, or pinned tabs. Tabs support activation, reorder, close, reopen/history behavior, and disambiguated names; avoid duplicate tabs for the same resource in a group. Support two editor groups with splitting and moving tabs between groups. Preserve focus and meaningful view-local state.

### 1C. Unified resource navigation
Explorer exposes Workbench-root and Collection resources through a compact hierarchical tree with resource icons, context actions, multi-selection, drag-and-drop, creation, and hierarchy-aware sorting through its header overflow menu. Explorer does not include resource-search or filtering controls. The dedicated Left Search panel handles resource and content discovery, while Command Center/Quick Open provides rapid navigation. Opening a Table selects its last-used View, falling back to its default View.

### 1D. Shared commands
Menus, toolbars, context menus, shortcuts, Command Palette, and contextual actions invoke a common command system with enablement and context. Shortcuts are customizable and conflicts are handled. Common commands remain discoverable.

### 1E. Contextual editing and feedback
Use inline editors, dialogs, dedicated editors, and the Right Panel according to the task. Bottom Panel includes Problems, Operations, and History surfaces. Notifications support preferences and actionable navigation. Preserve record editor, schema manager, View toolbar, creation flows, and dialog functionality already approved.

### 1F. Session and window restoration
Restore recent workbenches, windows, tabs, groups, panels, and layout state. A window shows one workbench or Home. Opening an already writable-open `.wkbn` activates its existing authority rather than creating another independent writer. New workbenches require name and destination before creation; successful operations autosave without an unsaved-workbench mode.

### 1G. Settings and accessibility
Centralize application and workbench settings with explicit scope. Provide keyboard-only workflows, meaningful focus/selection state, screen-reader context, accessible dialogs/menus, and WCAG 2.2 AA targets where applicable. Keep shell/session state distinct from canonical Core domain state.

## 2. Resource Model & Navigation — seven requirements

### 2A. Stable hierarchy
A Workbench contains non-nestable Collections, Tables, Queries, Dashboards, Automations, and workbench-root Files. Tables, Queries, Dashboards, and Automations may live at root or in a Collection. A Table owns Fields, Records, and Views; a Dashboard owns Widgets. Collections **organize** resources but do not own their underlying data. Deleting a Collection moves the Collection and all currently contained resources, including their genuinely owned descendants, to Trash as one recoverable cascading operation. Restoration preserves the original hierarchy and UUIDs, subject to Core dependency and conflict validation. Tables cascade deletion to their genuinely owned content.

### 2B. UUID identity and references
All persistent identity-bearing objects use stable UUIDs, including resources, Fields, Records, choices/options, Notes, and other applicable nested entities. Names, positions, labels, and locations never serve as identity. Renaming and moving preserve UUIDs and dependencies; duplication generates new UUIDs and remaps included references. Naming scopes, collisions, and ambiguous labels are handled consistently.

### 2C. Resource lifecycle
Support applicable create, rename, duplicate, move, import/export, and delete operations through shared Core rules. Explorer multi-selection and bulk actions obey ownership and dependency checks. Broken dependencies remain identifiable and actionable rather than being silently discarded.

### 2D. Table-owned Views
Each Table has a required default View and may own additional Views. Views configure presentation but do not duplicate Records. Opening a Table chooses the last-used valid View or default. Views can be renamed, duplicated, and managed within their Table.

### 2E. Navigation and selection
Explorer, Search, Quick Open, tabs, linked-record navigation, and contextual reveal use stable identities. View-local selection, scrolling, and navigation are distinct from saved View definitions. Revealing a Record must not silently rewrite saved filters; provide a clear navigation/reveal behavior when the Record is not currently visible.

### 2F. Record display labels
Record identity is UUID-backed. Tables support configurable single-Field or composite display labels and Linked Record Field-specific label overrides. Labels may be duplicated and are never used as references.

### 2G. Contextual Notes
Notes are supporting objects attached by UUID to Workbenches, Collections, Tables, Fields, Records, Views, Queries, Dashboards, Automations, or Files. Multiple Notes per target are allowed, with rich-text/Markdown editing, autosave, search, and contextual Right Panel management. **No Cell-level Notes**; comment on the Record when a particular value needs explanation. Notes are not standalone Explorer resources. Renaming/moving a target retains Notes; deletion and restoration follow target lifecycle. Standalone Markdown documents remain Files.

## 3. Tables, Fields & Relationships — six requirements

### 3A. Canonical structured data
Tables own Fields and Records; values conceptually map `(record UUID, field UUID) -> typed value`. A cell is presentation, not an independently owned object. Blank differs from zero, false, or an explicit empty value. Empty Tables may have zero Fields and Records; virtual blank Grid rows do not create Records until edited. Metadata and Notes belong to their proper owners. Schema changes preserve existing information where possible.

### 3B. Rich Field types and configuration
Support Text, Long Text, Email, Phone, URL; Number, Integer, Currency, Percentage, Rating; Boolean/Checkbox; Date, Time, DateTime, Duration; Single-choice and Multi-choice; Linked Record (single/multiple); Attachment; JSON; Formula, Lookup, Rollup/Summary; and Generated Fields. Choice options have stable identities. Currency has a configured currency; percentages use fractional underlying values; DateTime denotes an instant, Time a wall-clock time, Duration elapsed time. JSON remains structured content rather than implicit nested Fields. Display formats may share storage types. Each Field defines editor, formatting, defaults, constraints, and conversion behavior. No geographic or polymorphic multi-target relationship Field is required in V1.

### 3C. Calculated and generated values
Formula Fields use the shared typed expression language, stable Field bindings, and readable syntax such as `=[Quantity] * [Unit Price]` and supported relationship traversal. Lookup/Rollup/Summary Fields derive typed results through Linked Record relationships; supported aggregates include count, sum, average, min, max, and compatible extensions. Generated values include sequences, UUID, created/modified timestamps, and approved dynamic defaults such as random choice where appropriate. Generated and calculated Fields are normally read-only, with explicit Convert to Values where supported. Cycles are rejected; derived values are rebuildable and freshness-tracked.

### 3D. First-class relationships
Linked Record Fields point to one configured target Table and store target Record UUIDs, with single or multi-value cardinality. Support reverse navigation, Field-specific display labels, and candidate filtering from a Table or Query exposing stable source Record identities. Candidate restrictions do not silently erase pre-existing links. On target deletion, relationship policies support Preserve unresolved reference, Clear, or Restrict; relationship deletion does **not** automatically cascade to linked target Records. Queries, Lookups, Rollups, and Automations can consume relationships.

### 3E. Severity-based validation
Constraints include required, unique, typed/format, expression-based, and supported record-level rules. Rules may be **Errors** (block new commits) or **Warnings** (allow commit and appear in Problems). Structural/integrity rules always block invalid commits. Invalid interactive edits remain drafts until corrected or discarded. Never silently coerce or discard invalid input. Existing values made nonconforming by a schema change or legacy data are preserved and flagged; migration/repair is explicit. Bulk operations validate their proposed final state.

### 3F. Editing and schema management
Grid editing, Record Details, schema manager, multi-record creation, bulk edits, import, and Automations share Core operations. Support familiar spreadsheet navigation, selection, clipboard, virtual blank rows, and schema creation. Field type changes preview conversions, dependency effects, and incompatible values; preserve recoverable originals where needed. Defaults affect new Records, not retroactively all existing Records. Committed operations integrate with History, Undo/Redo, and dependency invalidation.

## 4. Views & Presentation — six requirements

### 4A. Independent saved Views
Each View presents one Table's canonical Records through independently persisted configuration: visible Fields, order, widths, frozen Fields, density, row height, filters, sorting, grouping, formatting, and appropriate view-specific settings.

### 4B. Five native View types
V1 includes **Grid, Kanban, Calendar, Gallery, and Pivot**. Each provides suitable navigation, selection, editing where applicable, and configuration. Pivot supports dimensions, aggregation, subtotals, and drill-through; Calendar uses date-bearing Fields; Kanban and Gallery use configurable cards.

### 4C. Organization and analysis
Support filters, multi-key sorting, multi-level **Field-based** grouping, collapsible groups, summaries, and manual Record ordering where applicable. Moving a Record between Field-derived groups updates the underlying Field value and clearly communicates that effect. Convenient Add Group actions may create/configure a visible underlying choice Field. **Do not implement separate manual Sections, Section membership, or independent nested Sections.**

### 4D. Formatting and rendering
Support Field-level formatting defaults, View overrides, conditional formatting, reusable formatting presets, and advanced declarative renderers. Formatting is safe, bounded, accessible, and does not mutate canonical values.

### 4E. Dashboards
Dashboards support responsive **12-column** widget layouts, placement/resizing, charts, KPIs, tables, text/labels, and approved widget variants. Widgets consume Tables, Views, or Queries as applicable. Support global and local filters, cross-filtering, drill-through, refresh/freshness states, and relevant interactive navigation. Broken sources remain identifiable in Problems.

### 4F. Consistent interactions
View toolbars, selection, keyboard behavior, contextual editors, linked-record navigation, and dependency updates follow shared semantics while respecting view-specific interaction models. UI renders bounded data windows rather than whole large Tables.

## 5. Queries & Expressions — six requirements

### 5A. Unified Query resources
One Query resource type supports either **Visual** or **SQL** definition mode, with UUID, lifecycle, dependencies, result schema, and execution status. A Query result is a read-only virtual dataset, not an editable Table.

### 5B. Visual Query Builder
Visual definitions use a typed application-owned Query Plan supporting source selection, compatible joins, projections, filters, computed expressions, grouping, aggregation, sorting, and limits. The builder can show generated SQL for inspection. Generated SQL is **not** editable as the source of truth for a Visual Query.

### 5C. Controlled native SQL
SQL Queries support authoring native SQLite `SELECT` queries, including supported joins, expressions, CTEs, subqueries, and aggregation, under enforced read-only access and resource bounds. SQL must not mutate internal SQLite state or access unauthorized internal metadata. **Duplicate as SQL Query** creates an independent editable SQL definition from supported generated SQL; no promise of two-way visual/SQL round-tripping.

### 5D. Composable dependencies
Queries can consume Tables and other Queries; cycles are rejected. Dependencies bind to stable identities, not display names. Results expose typed schemas, source Record identity where meaningful, and navigation to editable source Records. Query results with stable source Record identities may filter Linked Record candidates; aggregates without such identity may not.

### 5E. Typed expression semantics
The application-owned typed expression engine serves Formulas, Visual Query expressions, validation, conditional formatting, and Automation conditions/actions where applicable. Context-specific functions and limitations are explicit. SQL mode intentionally retains SQLite SQL semantics, not a fictional identical expression language.

### 5F. Bounded, observable execution
Execution pushes compatible work into SQLite, uses bounded streaming or specialized evaluation as necessary, and exposes progress, errors, result freshness, pagination, caching, and dependency status. Unbounded UI/application fallback is prohibited. Stale results are visibly identified.

## 6. Data Operations & Interoperability — six requirements

### 6A. Import/export
Import CSV, TSV, XLSX, and JSON into new Tables or existing Tables via append/upsert. Support preview, parsing, type inference, Field mapping, identity matching, conflict resolution, and validation. Export Table/View/Query data to supported interoperable formats, with selected/visible versus full-data options and clear derived-value semantics. Uploading a File is not importing its data.

### 6B. Structured clipboard
Support cell/range, Record/row, and Field/column copy/paste where meaningful. Internal clipboard data retains supported types, references, and metadata; external text/HTML formats interoperate with spreadsheet applications. Paste obeys schema, constraints, and transactional operations.

### 6C. Find, Replace & Transform
Provide scoped search/replace across selections, Records, Fields, Views, and Tables where applicable. Type-aware transformations support expressions, preview, and transactional application. Read-only derived Fields remain protected.

### 6D. Bulk operations
Support multi-record create/edit/delete/duplicate/move and applicable bulk schema/resource operations, with progress, cancellation when safe, meaningful History, and scheduled execution. Large operations must not block the UI.

### 6E. Atomic validation and explicit partial imports
Validate the full proposed final state and distinguish blocking Errors from Warnings. Imports may **explicitly commit a user-accepted valid subset atomically**; rejected rows remain available for correction/export. No rows are silently skipped. Upsert validation includes affected existing Records and cross-row constraints. Other operations obey their defined atomic boundaries and report failures without half-committed changes.

### 6F. Portable transfer
Support dependency-aware resource copy/export/import, identity remapping, associated Notes/Files, and explicit unresolved-dependency reporting. Native resource transfers use `.wkbn` files; ordinary tabular export and clipboard are separate operations.

## 7. Files, Portability & Templates — six requirements

### 7A. First-class Files
Files are workbench-root resources with UUIDs, metadata, rename/replace/delete operations, previews, and built-in editors for supported text/Markdown and structured formats. Preview images, PDFs, and other supported media; opening externally is available when appropriate. Attachment Fields refer to File UUIDs rather than duplicating bytes.

Files may be independent Workbench assets or attachment-managed assets referenced by one or more resources. Attachment associations use File UUIDs and do not duplicate File content. Ordinary detachment preserves the underlying File. Resource deletion offers a **Preserve attachments** checkbox, unchecked by default. If unchecked, attachment-managed Files referenced exclusively by the full deletion set are moved to Trash; if checked, they are preserved as independent Files. Shared Files referenced by surviving resources and intentionally independent Files are always preserved.

### 7B. Managed and external Files
Managed File bytes live within `.wkbn` through the logical File Store. External File resources retain paths/references and clearly show missing/inaccessible targets. Replacing content preserves the File resource UUID and appropriate dependencies. Embedded content is never implicitly executed.

### 7C. Dependency-aware resource transfer
Export selected resources with previewed dependencies, included schema/data/Views/relationships/Notes/Files, and choices for required versus optional dependencies. Import remaps UUIDs and resolves naming and dependency conflicts explicitly.

### 7D. Portable Workbench copies
Save a Copy and complete duplication produce independently usable native `.wkbn` SQLite files. When external Files exist, present an inclusion preview with **per-File choices and Include All**; excluded references remain explicitly external. Portability warnings cover missing or environment-dependent resources.

### 7E. Templates
Support complete-Workbench and appropriate resource templates with schema, presentation, dependencies, optional example data, and fresh identities on instantiation. Templates and transferred Automations obey trust activation rules.

### 7F. One native artifact format
A resource-transfer `.wkbn` is a valid Workbench database containing selected resources and dependencies, not a ZIP or separate native package extension. Compatibility checks, migration, missing-dependency reporting, and explicit trust/activation decisions apply.

## 8. Persistence, History & Recovery — six requirements

### 8A. Durable native persistence
`.wkbn` is the **live SQLite database**, with private versioned schema, structured resources, managed Files, and metadata. One writable Core authority per open workbench. Successful persistent operations commit their authoritative mutation and history durably; autosave is the norm. No browser database, account, or cloud dependency.

### 8B. Undo/Redo and History
Coherent semantic operations drive Undo/Redo and inspectable Record/schema/resource History. Bulk imports and Automations have meaningful attribution and transaction boundaries. Restoration uses Core rules and can surface conflicts. History is distinct from snapshots and Trash.

### 8C. Trash and deletion
Deleted resources and owned content are recoverable through Trash where applicable; permanent deletion is explicit. Table deletion cascades to its owned Fields/Records/Views and contextual Notes; linked target Records do not cascade merely through relationships. 

Deleting a Collection moves the Collection and all currently contained resources, including their genuinely owned descendants, to Trash as one recoverable operation. The operation requires an impact preview identifying affected resources and dependencies. Restoration preserves UUIDs and original Collection placement, subject to Core validation. Permanent deletion remains explicit. 

Deletion impact previews include attachment handling. The **Preserve attachments** checkbox defaults to unchecked. Attachment exclusivity is evaluated across the complete transactional deletion set, including cascading Collection and owned-resource deletions. Exclusively associated attachment-managed Files are included in Trash unless preservation is selected. Files referenced by surviving resources and intentionally independent Files are excluded from automatic attachment cleanup. Undo and restoration recover affected Files and associations consistently.

**No separate Archive lifecycle**; inactive content can use Collections and Explorer visibility. Restoration handles conflicts and dependencies.

### 8D. Backups and recovery
Support consistent SQLite backups while open, recovery checkpoints, configurable retention, safe restore, integrity diagnostics, and crash-safe interrupted-operation behavior. Backups are distinct from History, Undo/Redo, and Trash.

### 8E. Artifact versions and migrations
Explicit artifact/schema versions, compatibility checks, safe upgrades, backup-before-migration, and safe handling of newer unsupported files. Migrations preserve UUIDs and recoverable data, with explicit reporting of incompatible changes.

### 8F. Encryption and maintenance
Optional encrypted `.wkbn` storage protects canonical content and managed Files at rest through a supported SQLite encryption mechanism. Backup, portability, and migrations respect encryption. Provide integrity checks, optimization, cleanup, and diagnostic maintenance tools.

## 9. Execution & Performance — six requirements

### 9A. Bounded data access
Use bounded reads, pagination, virtualization, streaming, and data-layer sorting/filtering/grouping where appropriate. React never materializes entire large Tables or unbounded results. Preserve prior scale expectations: **1M Records routine, 10M primary V1 engineering target, 50M stress target, 100M+ best effort**; these are data/storage expectations, not simultaneous rendering guarantees.

### 9B. Operation scheduling
Coordinate interactive reads, serialized writes, Queries, imports, recalculation, backups, and maintenance. Prioritize user-facing work; provide progress, safe cancellation, and responsive UI. Background workers are execution helpers, never separate mutation authorities.

### 9C. Reactive freshness
Validated canonical writes commit transactionally. Derived Formulas, Lookups, Rollups, Queries, Views, and Dashboards update **asynchronously**, with explicit stale/fresh/error states. Older calculation generations must not overwrite newer results. Consumers requiring fresh data wait for the relevant dependency generation.

### 9D. Indexing and caching
Maintain identity/relationship/query indexes and rebuildable derived caches as useful. Optimization must not create a competing source of truth. Invalidation and recovery remain deterministic.

### 9E. Limits and isolation
Bound memory, execution time, result size, recursion, and expensive workloads. Failures remain local to affected calculations/operations and appear in Problems rather than corrupting canonical data.

### 9F. Operation visibility
Expose queued/running/completed/failed/canceled operations in Operations, Problems, notifications, and Status Bar, with actionable diagnostics and appropriate detail levels.

## 10. Automations — six requirements

### 10A. First-class resources
Automations have UUIDs, names, descriptions, enabled states, lifecycle operations, and root/Collection placement. They participate in Search, Explorer, duplication, templates, and resource transfer.

### 10B. Visual definitions
Provide declarative trigger–condition–action editing for supported data-change events, resource events, manual runs, and scheduled triggers. Conditions use the typed expression engine; actions use authorized Core operations. Arbitrary user scripts are not the Automation model.

### 10C. Deterministic execution
Core coordinates ordering, conflicts, transaction boundaries, bounded recursion/retriggering, and idempotence where appropriate. Automations cannot bypass validation, relationship policies, or write serialization.

### 10D. Trust and safety
Locally authorized Automations may resume according to saved settings. Imported, copied-in, templated, or otherwise untrusted Automations are **disabled until reviewed and explicitly enabled** in the local context. Authorization is not blindly transferred with a `.wkbn`. No arbitrary native code or direct internal SQLite mutation. Destructive/environment-dependent actions have explicit safeguards.

### 10E. Fresh dependencies and scheduling
Automations requiring derived data wait for fresh inputs or fail/report clearly. Scheduled Automations execute **only while the relevant Workbench Core is running**; no hidden always-on background service or cloud execution. Define missed-run behavior per Automation and surface it on reopen. Respect execution budgets and cancellation.

### 10F. Run history and recovery
Expose test/manual run, enable/disable, status, timestamps, affected resources, logs, failures, and supported Undo/History. Integrate with Problems, Operations, and notifications.

## V1 technical boundaries and exclusions

- **Desktop:** MōBrowser 2.x preferred; Electron fallback if a concrete blocker emerges. Runtime-specific APIs isolated from Core. Multi-window support respects single-writer ownership.
- **UI:** React + TypeScript; Workbench-owned Grid semantics with optional high-performance Grid adapter. DOM/Canvas/hybrid rendering is an implementation choice, not a domain contract.
- **Core:** Modular desktop monolith; one mutation authority, typed APIs, domain operations, application-owned expression/dependency/History semantics.
- **Data:** SQLite native `.wkbn`; private hybrid relational physical storage, with distinct logical structured repository and File Store.
- **No V1 requirement:** accounts, cloud sync, realtime multi-user collaboration, live cross-workbench links, arbitrary scripting or native-code Automations, polymorphic multi-Table references, spreadsheet-coordinate formulas, or a Field-plugin marketplace.
- **Not excluded:** native SQL SELECT Queries, five View types, advanced Dashboard interactions, optional encryption, templates, and Automations; earlier documents that excluded these are superseded.

## Acceptance invariants

1. Stable UUIDs, not labels/positions, determine identity.
2. All persistent mutations flow through Core and commit atomically with relevant History.
3. Severity-based validation blocks Errors but allows Warnings; existing nonconforming data is not silently destroyed.
4. Views and Query results never become independent copies of Table Records.
5. Collections organize; Tables own their nested content.
6. No manual Sections, no separate Archive lifecycle, and no second native package format.
7. `.wkbn` remains a live versioned SQLite database; transfers and copies remain valid `.wkbn` files.
8. React never directly accesses SQLite or requires unbounded data materialization.
9. Derived state is versioned/freshness-tracked and cannot silently appear current when stale.
10. Automations cannot bypass Core, validation, or local trust rules.
11. Accessibility, portability, and recoverability are architectural requirements.
12. Detailed engineering choices may be made during implementation without another pre-implementation validation phase.
