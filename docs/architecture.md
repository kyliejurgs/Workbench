# Workbench Architecture

Status: Working draft

## Purpose

This document records the technical architecture of Workbench, including confirmed decisions, their rationale, and questions requiring further evaluation.

The requirements document defines what Workbench must do. This document records how we have chosen to build it.

Workbench replaces Better Spreadsheet as a new application. Compatible product behavior may inform the design, but the existing codebase is not reused.

## Document Conventions

- Confirmed: An agreed architectural decision.
- Proposed: A recommended approach awaiting agreement.
- Tentative: A possible approach whose inclusion remains undecided.
- Open: A question requiring a decision.

A proposed or tentative approach is not a confirmed decision. Architecture decisions use sequential `ARCH-` identifiers. Where a newer architectural decision conflicts with the current requirements document, the discrepancy is identified for reconciliation rather than silently changing the requirements.

## 1. Application Platform

### Confirmed

- ARCH-01: Workbench V1 is a standalone desktop application built with Electron.
- ARCH-02: The desktop application supports Windows, macOS, and Linux.
- ARCH-03: The desktop interface uses React and TypeScript.
- ARCH-04: Workbench is a new application. Better Spreadsheet's existing implementation is not reused.
- ARCH-05: Desktop-specific capabilities remain separate from reusable application logic.
- ARCH-06: The Electron renderer accesses privileged application capabilities through a typed preload API and does not connect directly to SQLite.
- ARCH-07: Electron's main process coordinates application-level behavior and routes database operations to a dedicated utility-process data service.
- ARCH-08: Windows displaying the same workbench receive change notifications and reflect updates to shared underlying data.

### Rationale

Electron provides a cross-platform desktop shell using web technologies. Separating the renderer, main process, and data service establishes clear responsibilities, protects privileged capabilities, and keeps the interface independent of the storage implementation.

### Open

- Desktop packaging, distribution, signing, and update implementation.
- Precise IPC contracts, process recovery, and utility-process lifecycle.

## 2. Repository and Shared Core

### Confirmed

- ARCH-09: Workbench uses a monorepo containing the desktop application and reusable packages.
- ARCH-10: The reusable core is framework-independent TypeScript.
- ARCH-11: Data models, validation, formulas, and synchronization logic do not depend on React or Electron.
- ARCH-12: The architecture accommodates a future mobile companion without including mobile development in V1.

### Rationale

A monorepo allows coordinated development while preserving boundaries between the desktop application and shared code. A framework-independent core allows future clients to reuse business rules without inheriting desktop-specific implementation.

### Proposed

- Organize the repository around `apps/desktop` and `packages/core`, adding further packages when their boundaries become useful.

### Open

- Final package structure, names, and dependency boundaries.
- Build tooling and package management.

## 3. Workbench Storage and Boundaries

### Confirmed

- ARCH-13: SQLite is Workbench's authoritative local database.
- ARCH-14: Each workbench has its own SQLite database.
- ARCH-15: A small application-level SQLite registry tracks locally available workbenches and their locations.
- ARCH-16: Core editing works offline without requiring a server connection.
- ARCH-17: Each workbench can be backed up or exported independently.
- ARCH-18: File content is stored separately from SQLite, with file identities, metadata, and references tracked in the workbench database.
- ARCH-19: The data service uses a single writer, a bounded reader pool, and SQLite WAL mode where appropriate.
- ARCH-20: Operations that intentionally modify multiple related values use explicit database transactions.
- ARCH-21: Live record references and query dependencies stay within one workbench. Cross-workbench transfers do not create live references between workbenches.

### Rationale

Per-workbench databases provide isolation and portable backup boundaries. SQLite supports transactions and indexed queries without requiring a database server. Separate file storage avoids embedding all file content in database records.

### Open

- Database schema and migration strategy.
- Attachment and file layout.
- Atomic backup and restore procedures.
- Database and file recovery after interrupted operations.

## 4. Content Model and Organization

### Confirmed

- ARCH-22: V1 content includes tables, saved views, saved queries, saved dashboards, and first-class files.
- ARCH-23: Tables own their fields, records, and cell values. Fields and records have stable identities independent of their displayed names or positions.
- ARCH-24: A record is a row within a table rather than an independent top-level Explorer item.
- ARCH-25: Saved views belong to a table and present its underlying records without duplicating them.
- ARCH-26: Saved queries produce read-only derived datasets from tables or other saved queries within the same workbench.
- ARCH-27: Query dependencies cannot contain direct or indirect cycles.
- ARCH-28: Query results can supply data to dashboards.
- ARCH-29: Dashboards are saved, read-only presentations of table or query data. Dashboard widgets can navigate to their underlying source records, where editing occurs.
- ARCH-30: Files have stable identities and live in the workbench's central Files area. Other content can reference the same file without creating copies.
- ARCH-31: Documents, notes, and diagrams are file content rather than separate top-level content types.
- ARCH-32: Collections are optional, non-nesting organizational containers for tables, queries, and dashboards.
- ARCH-33: Each table, query, or dashboard belongs to at most one collection or remains ungrouped. Moving an item between collections does not change its identity or data.
- ARCH-34: Files remain in the central Files area rather than becoming collection members because another item references them.

### Rationale

Stable identities allow records and files to be reused across different presentations without duplication. Separating data ownership from organization prevents moving or renaming content from breaking its relationships.

### Open

- Reconcile the requirements document's nested-folder model with the newer non-nesting collections decision.
- Query authoring interface, execution strategy, and refresh behavior.
- Supported native file formats and their creation, preview, and editing capabilities.

## 5. Work Area and Windows

### Confirmed

- ARCH-35: The work area hosts multiple tabs containing different supported content types.
- ARCH-36: The active tab determines which content surface is currently displayed.
- ARCH-37: Opening content normally activates its existing tab. Users can explicitly open another tab for the same content when needed.
- ARCH-38: Multiple tabs can display the same table, with each tab independently selecting its active saved view.
- ARCH-39: All tabs and windows displaying a table share its underlying records. Edits propagate to other surfaces displaying the affected data.
- ARCH-40: Tables, views, and tabs have distinct responsibilities: a table owns data, a view defines a reusable presentation, and a tab hosts an open content surface.

### Rationale

The work area provides flexible multitasking without treating each open presentation as a separate copy of the data. Separating tabs from saved views allows the same table to be examined in different ways simultaneously.

### Open

- Detailed tab lifecycle, restoration, and state ownership.
- Coordination of view selection and record-detail panels across tabs and windows.

## 6. Spreadsheet Grid, Fields, and Formulas

### Confirmed

- ARCH-41: Prioritize free, commercially usable grid libraries.
- ARCH-42: Grid presentation remains separate from domain data, editing operations, and persistence.
- ARCH-43: V1 tables support spreadsheet-style keyboard navigation, range selection, multi-cell editing, row operations, fill and drag-to-fill, and column resizing and reordering.
- ARCH-44: V1 field types include text, numbers with currency and percentage formatting, boolean, date, date-time, single-select, multi-select, single-record reference, multi-record reference, attachment, and URL.
- ARCH-45: Field validation preserves and visibly flags invalid values rather than silently discarding them.
- ARCH-46: Blank values, zero, `false`, empty text, and errors retain distinct meanings.
- ARCH-47: Record references use stable identities within a workbench rather than relying on displayed values or row positions.
- ARCH-48: Column and cell formulas follow the requirements' calculated-field behavior, and dependency changes trigger recalculation.
- ARCH-49: Queries support selecting fields, filtering, sorting, joining data, and calculating derived values.
- ARCH-50: V1 dashboards support data grids, summary values, and basic charts.

### Rationale

The grid provides familiar interactions without defining the underlying data model. Stable identities, explicit validation states, and independent calculation logic allow tables to support multiple views, queries, and dashboards consistently.

### Open

- Grid library or rendering foundation.
- Performance, keyboard interaction, and accessibility validation.
- Formula syntax, addressing, and function catalog.
- Detailed field conversion behavior.
- Query evaluation and dependency invalidation strategy.

## 7. Editing and Durability

### Confirmed

- ARCH-51: The UI may apply optimistic edits, but an edit is durable only after its SQLite transaction commits successfully.
- ARCH-52: Pending, confirmed, and failed edit states must be distinguishable.
- ARCH-53: Failed edits must not silently appear saved and should remain recoverable or retryable where practical.
- ARCH-54: Normal cell edits are field-level writes.
- ARCH-55: Concurrent edits to the same field from local windows resolve by the last successfully committed local edit, with change notifications sent to other windows.
- ARCH-56: Intentional multi-field and multi-record operations commit transactionally rather than exposing partially completed changes.

### Rationale

The SQLite commit establishes a clear durability boundary. Optimistic presentation keeps editing responsive without misrepresenting whether a change has been saved.

### Open

- Retry and recovery interface for failed edits.
- Detailed operation contracts and error propagation between renderer and data service.

## 8. Undo and Redo

### Confirmed

- ARCH-57: A shared in-memory operation journal records committed user operations.
- ARCH-58: Undo history is associated with the underlying edited items rather than individual visual tabs.
- ARCH-59: Multi-item operations are undone and redone atomically across their affected histories.
- ARCH-60: New edits invalidate redo history for affected items and linked operations.
- ARCH-61: The shared operation journal is cleared when the application fully exits.
- ARCH-62: Active-cell editing maintains temporary local undo and redo stacks.
- ARCH-63: Deactivating a cell clears its temporary editing stacks.
- ARCH-64: After local editing history is exhausted, the undo or redo shortcut exits uncommitted editing and delegates to the shared operation journal.

### Rationale

Item-based history remains consistent across different views of the same data. Temporary cell-editing history allows text-level undo without confusing it with committed changes to workbench content.

### Open

- Reconcile the requirements document's persistent, workbench-wide undo history with the newer in-memory, item-scoped architecture.
- Operation-journal representation and handling of linked histories.
- Exact history retention and memory limits.

## 9. Clipboard and Cross-Workbench Transfers

### Confirmed

- ARCH-65: Clipboard exchange interoperates with Excel and Google Sheets.
- ARCH-66: Workbench-to-Workbench moves coordinate source and destination operations.
- ARCH-67: Pasting a cut selection into an external application does not automatically delete the Workbench source because successful receipt cannot be verified.
- ARCH-68: Moving records within the same workbench preserves their identities and references.
- ARCH-69: Copying records creates new identities.
- ARCH-70: Cross-workbench transfers create destination identities and cannot establish live references back to the source workbench.
- ARCH-71: References between records copied together can be remapped to their corresponding destination records.
- ARCH-72: Unresolved cross-workbench references retain enough information for review. Suggested replacements require explicit user confirmation.
- ARCH-73: Required references need valid replacements; optional unresolved references may be cleared.
- ARCH-74: Cross-workbench moves use a durable transfer journal separate from the in-memory undo journal.
- ARCH-75: Recovery verifies completed transfer steps. If completion is uncertain, the source is preserved, affected rows are protected, and the user can resolve the transfer without blocking unrelated data.

### Rationale

Identity-preserving moves protect relationships within a workbench. Cross-workbench transfers require new identities because workbenches are isolated. A durable transfer journal prevents interrupted moves from silently losing records.

### Open

- Exact clipboard formats and metadata.
- Formula translation and paste-conflict behavior.
- Transfer-journal schema and recovery interface.
- Matching and reference-repair interface.

## 10. Search

### Confirmed

- ARCH-76: Global search defaults to the active workbench.
- ARCH-77: Users can switch global search to All Workbenches to search across all locally available workbenches, including offline.
- ARCH-78: Search results identify their source workbench and open the matching item or record in the appropriate workbench.

### Rationale

Search remains useful offline and respects workbench boundaries while allowing users to locate content across their locally available workbenches.

### Open

- Whether cross-workbench search queries each local database or uses a separate application-level index.
- Search indexing, ranking, and performance strategy.

## 11. Synchronization

### Confirmed

- ARCH-79: Cross-device synchronization is part of V1.
- ARCH-80: Local editing remains available without a network connection.
- ARCH-81: V1 uses a hosted synchronization service.
- ARCH-82: Each device retains its local SQLite database. Hosted synchronization exchanges changes rather than replacing local-first storage.
- ARCH-83: A hosted service's free tier is acceptable, but a paid plan must not be required for the intended personal-use experience.
- ARCH-84: Full-workbench backup and export do not depend on the synchronization provider.
- ARCH-85: Concurrent offline changes to different fields of the same record can merge.
- ARCH-86: Conflicting offline edits to the same field preserve both candidate values and require user resolution rather than silently discarding either value.

### Rationale

Hosted synchronization provides a common exchange point while allowing workbenches to remain useful offline. Explicit same-field conflict resolution protects user work when devices make incompatible changes independently.

### Open

- Reconcile the requirements document's automatic same-cell conflict selection with the newer explicit user-resolution decision.
- Hosting provider and synchronization protocol.
- Authentication and account management.
- Change ordering and identification across offline devices.
- Attachment synchronization and offline availability.
- Conflicts involving deletions, schema changes, files, and other non-cell content.
- Recovery from interrupted synchronization and provider quota limits.

## 12. Costs and Dependencies

### Confirmed

- ARCH-87: Prioritize free, commercially usable software dependencies.
- ARCH-88: Workbench V1 must be possible to build and personally use without required ongoing payments.

### Rationale

Core application capabilities should not depend on paid-only library features or a mandatory ongoing subscription.

### Open

- Provider free-tier capacity and commercial-use terms.
- Optional paid services or components that may be supported without becoming V1 requirements.
- Behavior when third-party free-tier terms change or hosted quotas are exhausted.

## 13. Architectural Principles

### Confirmed

- ARCH-89: Keep the user interface, reusable business logic, local persistence, and synchronization responsibilities separate.
- ARCH-90: Preserve the data portability and recovery behaviors defined in the requirements.
- ARCH-91: Avoid desktop-only dependencies in code intended for reuse by a future mobile client.

These principles guide later decisions but do not prescribe implementations that have not yet been evaluated.

## 14. Implementation Sequence

### Proposed

- Establish the monorepo, Electron shell, typed preload boundary, and utility-process data service.
- Implement the application registry, per-workbench SQLite persistence, migrations, and file identities.
- Build tables, saved views, multi-tab and multi-window updates, and durable editing.
- Implement formulas, references, clipboard, and undo behavior against shared domain operations.
- Add saved queries, read-only dashboards, the central Files area, and collections.
- Integrate hosted synchronization and explicit conflict resolution.
- Complete backup and restore, packaging, cross-platform testing, and recovery testing.

### Open

- Reconcile the identified requirements conflicts before treating the architecture and requirements documents as a single authoritative development baseline.
- Select the grid foundation and hosted synchronization provider and protocol.
- Complete detailed feature specifications without reopening confirmed architecture decisions.
