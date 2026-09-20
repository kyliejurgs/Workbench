# Workbench Requirements

Status: Working draft

## Purpose

Workbench is a standalone, cross-platform data management and visualization application built around connected, reusable datasets.

Users primarily manage data through a structured spreadsheet interface. Saved views, dashboards, and automations provide additional ways to organize, understand, and act on that data.

Example uses include game collections, quests and achievements, inventory, recipes, and ordering. These scenarios inform the design without limiting the product to those uses.

## Document Conventions

- Confirmed: An agreed product requirement.
- Proposed: A suggested behavior awaiting agreement.
- Tentative: A possible capability whose inclusion remains undecided.
- Open: A question requiring a decision.

Confirmation establishes an agreed product requirement, not necessarily a V1 deliverable. Section 15 identifies the confirmed V1 scope. Requirements outside V1 remain part of the product direction unless explicitly revised.

## 1. Workbenches and Organization

### Confirmed

- ORG-01: Users can create and name multiple independent workspaces, each called a workbench.
- ORG-02: Each workbench contains its own datasets, views, dashboards, and other content.
- ORG-03: Content is organized hierarchically through nested folders.
- ORG-04: Data is reusable throughout its workbench.
- ORG-05: Workbenches are isolated. References, formulas, and dashboards cannot access data in another workbench.
- ORG-06: Moving content within a workbench preserves its references and relationships.
- ORG-07: Users can rename, duplicate, and delete datasets.
- ORG-08: Users can create, rename, move, and delete folders and other supported workbench items.
- ORG-09: The explorer supports drag-and-drop organization.

## 2. Application Experience

### Confirmed

- APP-01: Workbench is a standalone, cross-platform application.
- APP-02: Navigation and layout follow VS Code-style conventions by default, with specific exceptions documented as needed.
- APP-03: A sidebar explorer displays the active workbench's hierarchy.
- APP-04: Datasets, dashboards, and other content open in tabs.
- APP-05: Users can split the work area into tab groups to view content side by side.
- APP-06: Users can move tabs between groups and resize groups.
- APP-07: Data edits save automatically.
- APP-08: Reopening a workbench restores its open tabs and layout.
- APP-09: Global search defaults to the active workbench and searches its locally available content.
- APP-10: Scoped search limits results to a selected context, such as a folder, dataset, or current view.
- APP-11: The active search scope is clearly displayed.
- APP-12: Workbench supports Windows, macOS, and Linux.
- APP-13: Users can create and use local workbenches without an account.
- APP-16: Users can open multiple workbenches simultaneously in separate windows.
- APP-17: Each window displays one workbench and maintains that workbench's tabs and layout.
- APP-18: When opening a workbench, users can choose to use the current window or a new window.
- APP-19: Users can open the same workbench in multiple windows, each with its own tabs and layout.
- APP-20: Windows displaying the same workbench share underlying data, and edits are reflected across those windows.
- APP-21: Users can open a record detail view in a right-side panel without leaving the current spreadsheet or view.
- APP-22: The record detail panel displays fields, references, attachments, notes, and comments for the selected record.
- APP-23: Users can edit record details in the panel according to their permissions. Changes update the same underlying record.
- APP-24: Users can resize or hide the record detail panel.
- APP-25: The record detail panel follows the selected record by default.
- APP-26: Users can pin the displayed record so the panel remains on it while selection changes elsewhere.
- APP-27: Unpinning returns the panel to the currently selected record and resumes following selection.
- APP-28: Each completed user action that changes workbench content or organization is recorded as one undoable and redoable operation, except explicitly irreversible actions and actions that exceed the undo storage budget after a warning.
- APP-29: All changes made by a single action, such as a bulk paste or multi-item move, are undone or redone together.
- APP-30: Undo/redo excludes exports, sharing and permission changes, permanent deletion, and emptying trash. Recovery through retained snapshots remains separate.
- APP-31: Undo cannot cross the point where an edit from another user is applied through synchronization.
- APP-32: Undo/redo history is bounded by a retention limit. Actions beyond that limit are unavailable.
- APP-33: A synchronized edit from another user establishes an undo boundary across the entire workbench, regardless of which dataset or item was changed.
- APP-34: Undo/redo history persists across application restarts, subject to the history limit and workbench-wide synchronization boundary.
- APP-35: A user's own edits remain undoable and redoable across their devices, subject to the history limit and boundaries established by other users' synchronized edits.
- APP-36: Synchronizing edits from another device belonging to the same user does not establish an undo boundary.
- APP-37: The undo/redo history limit is measured in completed user actions, not elapsed time. Each bulk action counts as one action.
- APP-38: Undo/redo history defaults to 100 completed user actions.
- APP-39: Users can configure the maximum number of retained undo/redo actions.
- APP-40: Each workbench has a configurable storage budget for undo/redo history.
- APP-41: Undo/redo history is limited by the configured action count and storage budget. When either limit is exceeded, remove the oldest complete actions, except for the temporary oversized-action allowance defined in APP-43 through APP-47.
- APP-42: The interface indicates when storage constraints reduce history below the configured action count.
- APP-43: If a single completed action exceeds the undo storage budget, retain its complete undo information temporarily so the action can still be undone.
- APP-44: Undo/redo action limits and storage budgets are user-configurable preferences for each workbench. Changing them does not affect other users or override the synchronization boundary.
- APP-45: The next completed edit removes the oversized action from undo/redo history. Other users' synchronized edits still enforce the existing workbench-wide boundary.
- APP-46: Undoing or redoing an oversized action does not expire its temporary storage exception. It remains available for undo/redo until the next completed new edit or an applicable synchronization boundary.
- APP-47: Retaining an oversized action clears older undo/redo history, leaving the oversized action as the sole retained entry.
- APP-48: Opening another item in a tab or split group is the normal multitasking workflow. Opening a separate application window requires an explicit Open New Window command.
- APP-49: Workbench restores the previous windows and each window's tabs and pane layout when reopened.
- APP-50: Users can switch global search to All Workbenches to search item names and record field values across all locally available workbenches, including while offline.
- APP-51: Global search results identify their source workbench, can be filtered by dataset and field type, and can be opened directly. Opening a result from another workbench opens that workbench and navigates to the matching item or record.
- APP-52: Workbench supports light mode, dark mode, follow-system appearance, adjustable interface scale, and accent-color selection.
- APP-53: Application styling uses a centralized design-token system.
- APP-54: Workbench provides built-in update checks and downloads. Users choose when to install updates.
- APP-55: Updates preserve local data, handle migrations safely, and do not interrupt an active work session.
- APP-56: Workbench supports keyboard navigation throughout the app, visible focus indicators, scalable text, sufficient contrast, accessible labels, and reduced-motion support.
- APP-57: Spreadsheets, card views, dashboards, and dialogs support screen-reader use.

### Tentative

- APP-14: A future mobile companion may provide access to workbench data through an interface suited to phones or tablets. Mobile is not currently a committed release deliverable.

### Architectural Consideration

- APP-15: Data formats, synchronization, and core rules should accommodate a future mobile client without depending exclusively on desktop functionality.

### Open

- Detailed keyboard, navigation, and panel behavior.
- Define the default undo/redo storage budget and supported configuration limits.

## 3. Datasets and Fields

### Confirmed

- DATA-01: Users can create datasets and define their fields.
- DATA-02: The spreadsheet interface is the primary way to create, edit, and organize records.
- DATA-03: Multiple workflows and views can use the same underlying dataset without requiring duplicate copies.
- DATA-04: Fields define the types of values they hold.
- DATA-05: Values that violate field definitions or validation rules are retained and visibly flagged.
- DATA-06: Users can optionally require a field to contain a value. Missing required values are flagged as invalid.
- DATA-07: Users can optionally require unique values within a dataset's column. Duplicates are retained and flagged as invalid.
- DATA-08: Attachment fields support multiple files per cell.
- DATA-09: Users can open or remove attachments individually.
- DATA-10: URL fields store web addresses that users can open as links.
- DATA-11: Workbench is local-first. Core functionality operates on locally stored data without requiring a server connection.
- DATA-12: Each workbench has a shared attachment library that stores its files.
- DATA-13: Attachment cells reference library files rather than storing separate copies. The same file can be referenced by multiple records.
- DATA-14: Users can upload new files or select existing library files when adding attachments.
- DATA-15: Removing an attachment from a cell removes that reference without deleting the library file or other references to it.
- DATA-16: Uploading a file with identical contents to an existing library file reuses that file, even if the filename differs. Deduplication is scoped to the workbench.
- DATA-17: Attachment access respects sharing permissions. Access to an attached file does not grant access to unrelated library files.
- DATA-18: Deleting a referenced library file warns the user and allows them to proceed. The file enters trash under the standard retention rules.
- DATA-19: References to deleted library files retain their displayed file information and show a broken-reference indicator.
- DATA-20: Restoring a library file reconnects references that have not been removed or reassigned.
- DATA-21: Users can copy and paste rectangular ranges of cells within Workbench and exchange tabular clipboard data with applications such as Excel and Google Sheets.
- DATA-22: Pasted values follow the destination fields' validation rules. Invalid values are retained and visibly flagged.
- DATA-23: Copying cells preserves their values, formulas, and available type information.
- DATA-24: Pasting into an empty range attempts to resolve and preserve source formulas. If a formula cannot be resolved, paste its available calculated value with a broken-formula indicator showing that it does not recalculate.
- DATA-25: Paste Values Only ignores source formulas and pastes their available calculated values, subject to destination validation and formula-protection rules.
- DATA-26: Pasting over populated cells preserves incoming data and compatible type information wherever possible, subject to destination field definitions and validation.
- DATA-27: When pasting over a formula-controlled cell, an equivalent incoming formula does not create a formula conflict. The destination formula remains active and calculates its result.
- DATA-28: Before applying a paste that conflicts with existing formulas, present the affected cells and their existing and incoming content for review.
- DATA-29: Existing formulas remain unchanged until the user explicitly resolves the conflicts. Users can cancel without modifying the destination.
- DATA-30: Paste-conflict resolution offers Keep Existing Formula, Use Incoming Formula when valid, and Cancel.
- DATA-31: An incoming formula that overrides a column formula is visibly marked as an override.
- DATA-32: Paste-conflict resolution does not allow replacing an active formula with a static value, including through Paste Values Only.
- DATA-33: Users can add, rename, reorder, delete, and change the type of fields.
- DATA-34: Users can add, duplicate, delete, move, and copy records between compatible datasets.
- DATA-35: Changing a field's type previews conversion issues. Values that cannot be converted are preserved and visibly flagged.
- DATA-36: Spreadsheet editing supports keyboard navigation and shortcuts, range selection, multi-cell editing, row operations, fill handle, drag-to-fill, and column resizing and reordering.
- DATA-37: Field-level validation supports number ranges, text-length limits, allowed values, and date limits in addition to required and unique fields.
- DATA-38: Attachment fields support image previews and display download and synchronization status.
- DATA-39: Attachments in synchronized workbenches synchronize across devices.

### Confirmed V1 Field Types

- Text
- Number, with decimal, currency, and percentage formatting
- Boolean
- Date
- Date and time
- Single select
- Multi-select
- Single-reference
- Multi-reference
- Attachment
- URL

### Open

- Detailed field-type behavior and formatting.
- Record labels and identification in the interface.
- Treatment of empty values in unique fields.
- Attachment size limits and offline availability.
- Define formula equivalence after reference translation.

## 4. References

### Confirmed

- REF-01: Reference fields link to records within the same workbench.
- REF-02: A single-reference field links each cell to one record.
- REF-03: A multi-reference field links each cell to multiple records.
- REF-04: The field definition determines reference cardinality.
- REF-05: References remain valid when records are renamed, sorted, or moved within the spreadsheet view.
- REF-06: Deleting a referenced record warns the user that references will break and allows the user to proceed.
- REF-07: Broken references preserve their last displayed value and show a broken-reference indicator.
- REF-08: Multi-reference cells identify individual broken entries.
- REF-09: Restoring a deleted record reconnects references that have not been removed or reassigned.
- REF-10: Each reference field targets one specified dataset. Single-reference cells select one record, and multi-reference cells select multiple records from that dataset.
- REF-11: Users can choose the field used to display records in reference cells.
- REF-12: References identify records independently of their displayed values, so changing a display value does not break the link.
- REF-13: Users can open a referenced record directly from a reference cell to inspect or edit it according to their permissions.
- REF-14: Multi-reference cells allow users to open each referenced record individually.
- REF-15: Users can search records when selecting values for a reference field.
- REF-16: Users can inspect linked-record details without losing their current context.
- REF-17: Users can see which records reference the current record.

### Tentative

- Reference fields that can target multiple datasets.

### Open

- Detailed reference picker and inspection interactions.
- How dependency information is presented before deletion.

## 5. Formulas and Calculations

### Confirmed

- CALC-01: Users can define formulas for an entire column or an individual cell.
- CALC-02: Column formulas apply to existing and newly added rows.
- CALC-03: An individual cell formula can override its column formula.
- CALC-04: Overridden cells are visibly distinguishable from cells using the column formula.
- CALC-05: Users can inspect an override and restore the column formula.
- CALC-06: Manually entered values cannot override an active calculated cell.
- CALC-07: Formulas can use data from connected datasets.
- CALC-08: Formula results follow the field's type rules.
- CALC-09: Calculations update when their dependencies change.
- CALC-10: When a reference breaks, preserve the last-known values needed by dependent calculations.
- CALC-11: Formulas using broken references calculate with retained values and display an error indicator alongside the result.
- CALC-12: Formulas depending on invalid input values display an explanatory error instead of a calculated result.
- CALC-13: Dependency errors propagate to downstream formula cells.
- CALC-14: Restoring a referenced record triggers recalculation and clears errors where their cause has been resolved.
- CALC-15: V1 formulas support arithmetic, comparisons, text joining, conditional logic, and common aggregate functions.
- CALC-16: V1 formulas support date/time functions, text manipulation, rounding, and null/error handling.
- CALC-17: Formulas can aggregate values across multi-reference fields.

### Open

- Formula syntax, addressing, and exact function catalog.
- Circular dependency behavior.
- Details of retained dependency data after deletion.
- Formula behavior after structural changes.

## 6. Saved Views

### Confirmed

- VIEW-01: Users can create multiple saved views of a dataset.
- VIEW-02: Views share the same underlying records.
- VIEW-03: Each view maintains its own filters, sorting, and visible columns.
- VIEW-04: Editing through a view updates the underlying dataset.
- VIEW-05: Views support grouping records by a field.
- VIEW-06: Groups support summaries such as counts, totals, and averages.
- VIEW-07: Grouping and summary settings are saved with the view.
- VIEW-08: V1 supports spreadsheet views and card views.
- VIEW-09: Spreadsheet views save column order, widths, and per-view formatting in addition to their existing settings.
- VIEW-10: Card views allow users to choose a card title, image, and displayed fields.
- VIEW-11: Card views support filtering, sorting, and grouping.
- VIEW-12: Opening a card provides access to the underlying record's details for editing.

### Open

- Additional view types beyond spreadsheet and cards.
- Nested grouping.
- How summaries represent invalid or stale calculated values.

## 7. Dashboards

### Confirmed

- DASH-01: Users can create dashboards with customizable widgets.
- DASH-02: Widgets include charts, statistic cards, and filtered tables.
- DASH-03: A dashboard can draw from multiple datasets in its workbench.
- DASH-04: Users can arrange and resize widgets.
- DASH-05: Widgets reflect changes to their underlying data.
- DASH-06: Widgets can define their own filters and summary settings.
- DASH-07: Users can define shared dashboard filters and choose which widgets they affect.
- DASH-08: Widget filters combine with applicable shared filters.
- DASH-09: Widgets can operate independently of shared filters.
- DASH-10: Users can click through from dashboard widgets to inspect the underlying records.

### Open

- Initial chart and widget types.
- Mapping shared filters across different datasets.
- Display of invalid, stale, or inaccessible source data.

## 8. Offline Use and Synchronization

### Confirmed

- SYNC-01: Users can create, edit, and view locally available workbench content without an internet connection.
- SYNC-02: Changes save locally and queue for synchronization.
- SYNC-03: Pending changes synchronize automatically when connectivity returns.
- SYNC-04: The interface shows pending changes, sync progress, and failures.
- SYNC-05: Synchronization supports a user's multiple devices and authorized participants in shared content.
- SYNC-06: Synchronization merges changes independently at the cell level. When the same cell has conflicting edits, the most recently edited value becomes the synchronized value.
- SYNC-07: Displaced conflicting values are retained temporarily for recovery rather than silently discarded.
- SYNC-08: Signing in is required for synchronization and online sharing, but not for local use.
- SYNC-09: Synchronization is enabled separately for each workbench, allowing users to keep some workbenches entirely local while synchronizing others.
- SYNC-10: Disabling synchronization for a workbench on a device preserves its local data and allows local editing to continue.
- SYNC-11: Disabling synchronization affects only that device. It does not delete the server copy, stop synchronization on other devices, or change other users' access.
- SYNC-12: Deleting server content and revoking shared access are separate, explicit actions.
- SYNC-13: Enabling or resuming synchronization applies the same merge rules to local and server changes.
- SYNC-14: Conflicts with ambiguous edit ordering, deletions, or incompatible structural changes require user resolution when they cannot be merged safely.
- SYNC-15: V1 synchronization supports one user's workbenches across multiple computers. Multi-user collaboration is deferred, but synchronization must be designed to accommodate it.
- SYNC-16: Users can review and resolve conflicts that cannot be merged safely.
- SYNC-17: Signing out stops synchronization but preserves access to locally downloaded workbenches.
- SYNC-18: Removing local workbench data from a device is a separate, explicit action that warns about unsynchronized changes.

### Open

- Authentication methods and account management.
- Reliable ordering of edits from offline devices with inaccurate clocks.
- Retention and recovery interface for displaced conflicting values.
- Merge rules for non-cell content, including dashboards, folders, and automation definitions.
- Sync infrastructure and storage.
- Conflicts involving deletion or structural changes.
- Offline attachment availability.
- Handling access revocation while a device is offline.

## 9. Sharing and Permissions

### Confirmed

- SHARE-01: Users can share workbenches with other users.
- SHARE-02: Access supports Owner, Editor, and Viewer roles.
- SHARE-03: Users can share individual folders, datasets, dashboards, and other content without sharing the entire workbench.
- SHARE-04: Sharing an item does not grant access to unrelated content.
- SHARE-05: Sharing a dashboard prompts the user to choose whether to include access to its source datasets.
- SHARE-06: With source dataset access, a dashboard can remain live and reflect subsequent changes according to permissions.
- SHARE-07: Without source dataset access, sharing creates a view-only snapshot preserving the item's current state and the data it uses.
- SHARE-08: Sharing snapshots do not receive later source updates.
- SHARE-09: Users can revoke previously granted access.
- SHARE-10: Revocation prevents further server access and synchronization in either direction for the revoked content.
- SHARE-11: Revocation does not delete local copies, received snapshots, or exports.
- SHARE-12: The sharing interface explains the limits of revocation.
- SHARE-13: Access granted to a workbench or folder is inherited by its contents, including nested folders and newly created items.
- SHARE-14: Inherited access uses the role granted at the parent level unless an explicit permission exception applies.
- SHARE-15: Authorized users can customize access for individual folders and items, overriding inherited permissions.
- SHARE-16: Permission overrides can grant a different role or explicitly deny access.
- SHARE-17: The sharing interface shows each user's effective access and whether it is inherited or explicitly assigned.
- SHARE-18: Removing an override restores inherited access.
- SHARE-19: The workbench owner always retains full access to all content within the workbench. Item-level permission overrides cannot restrict or deny the owner's access.
- SHARE-20: Sharing access is a separate permission from Owner, Editor, and Viewer roles.
- SHARE-21: The owner manages sharing by default and can explicitly grant sharing access to other users for specified content.
- SHARE-22: Editor or Viewer access alone does not permit resharing content or changing its sharing permissions.
- SHARE-23: Users with sharing access can grant recipients no more access than they themselves hold, and only for content they are authorized to share.
- SHARE-24: Users with sharing access can grant sharing access to recipients for content they are authorized to share. Recipients remain subject to the same access-level and scope limits.
- SHARE-25: Revoking a user's access or sharing permission does not automatically revoke access they previously granted to others. Those grants must be revoked explicitly.
- SHARE-26: The owner can view a central access list covering all workbench content, including access granted by other sharers.
- SHARE-27: The access list identifies each recipient, their role, sharing permission, content scope, and the source of their access.
- SHARE-28: The owner can explicitly revoke selected access grants from this list.
- SHARE-29: An explicit No Access restriction takes precedence over other access grants, except for the workbench owner's guaranteed access.
- SHARE-30: The owner can assign permissions that override a recipient's access granted by other sharers.
- SHARE-31: Other sharers cannot bypass or weaken owner-assigned restrictions by issuing additional grants.

### Open

- Define precedence when multiple access grants and explicit restrictions overlap.
- Ownership transfer.
- Access to indirect dataset dependencies.
- How individually shared content appears to recipients while preserving workbench isolation.

## 10. Recovery Snapshots

Recovery snapshots restore workbench state. They are distinct from view-only snapshots created for sharing.

### Release Scope

Recovery snapshots are confirmed future capabilities, not V1 deliverables. V1 includes autosave, persistent undo/redo, trash, and recovery of values displaced by sync conflicts. Storage and change tracking must allow recovery snapshots to be added later.

### Confirmed

- HIST-01: Recovery snapshots cover the entire workbench.
- HIST-02: Users can create manual snapshots.
- HIST-03: Workbench also creates automatic snapshots under a policy that remains to be defined.
- HIST-04: Snapshots capture data, dataset definitions, field types, formulas, validation rules, saved views, dashboard configuration, and folder organization.
- HIST-05: Restoring a snapshot returns included content to its captured state, reversing subsequent additions, edits, and deletions.
- HIST-06: Items permanently deleted after capture can return when restoring a snapshot that contains them.
- HIST-07: Snapshots do not restore sharing permissions, access grants, personal application preferences, or window layout.
- HIST-08: Recovery snapshots preserve the attachment files needed to restore their captured state. Unchanged file contents are shared across snapshots without duplicate storage.

### Proposed

- Allow users to name manual checkpoints.
- Automatically snapshot before major destructive or structural changes.
- Create periodic recovery snapshots while changes are being made.
- Preserve the current state before restoring an older snapshot.
- Keep manual checkpoints until explicitly deleted and prune automatic snapshots according to a retention policy.

### Open

- Automatic triggers, frequency, and retention.
- Snapshot coverage of comments, trash, and automation state.
- Permissions and coordination for restoring a shared workbench.
- Snapshot storage limits.

## 11. Trash

### Confirmed

- TRASH-01: Individually deleted items enter trash.
- TRASH-02: Trash retention defaults to 30 days.
- TRASH-03: Users can restore items before retention expires.
- TRASH-04: Restoring an item cancels its deletion countdown.
- TRASH-05: Deleting a restored item starts a new full retention period.
- TRASH-06: Restored items retain their identity so eligible broken references reconnect automatically.
- TRASH-07: Users can permanently delete selected trashed items with confirmation.
- TRASH-08: Users can empty the entire trash with confirmation.
- TRASH-09: Confirmation explains that items will no longer be recoverable through trash.
- TRASH-10: Items are permanently removed from trash after retention expires.
- TRASH-11: Removing items from trash does not rewrite existing snapshots.
- TRASH-12: Users can configure trash retention.
- TRASH-13: Users can disable automatic permanent deletion from trash.
- TRASH-14: Deletion and restoration of synchronized items propagate across devices.

### Open

- Which item types have independent trash entries.
- How retention expiry is processed across offline devices.

## 12. Automations and Extensibility

### Confirmed

- AUTO-01: Users configure automations through a visual editor.
- AUTO-02: Automations support triggers, optional conditions, and actions.
- AUTO-03: Automations support event-based and scheduled triggers.
- AUTO-04: Schedules support specified times and recurring intervals.
- AUTO-05: Due automations execute while the application is running.
- AUTO-06: Each scheduled automation has a configurable missed-run policy: skip missed runs, run once when Workbench reopens, or catch up on every missed occurrence.
- AUTO-07: Catch-up execution is subject to safeguards against excessive or duplicate runs.
- AUTO-08: Automations operate within their workbench and respect permissions.

### Tentative

- Custom expressions or scripts.
- Installable extensions adding new capabilities.

### Open

- Exact V1 trigger, condition, and action catalog.
- Duplicate-run prevention across devices and collaborators.
- Failure reporting, retries, and loop prevention.
- Whether a workbench must be open for its automations to execute.
- Extension mechanisms and release scope.

## 13. Import and Export

### Confirmed

- IO-01: Users can import CSV and Excel files.
- IO-02: Users can export CSV, Excel, PDF, and native Workbench files.
- IO-03: Native exports support a complete workbench or selected content with the dependencies required to use it.
- IO-04: Native exports preserve included structure, relationships, formulas, and configuration.
- IO-05: Importing a complete native workbench creates a new workbench.
- IO-06: Importing smaller native content lets users choose the current workbench or a new workbench as its destination.
- IO-07: Import name conflicts require a user choice before applying the affected import.
- IO-08: Native exports include attachment files by default. Selected-content exports include only attachments used by the exported content.
- IO-09: Users can exclude attachment files from a native export, with a clear indication that the export will not contain those files.
- IO-10: V1 supports importing CSV and Excel files into datasets.
- IO-11: V1 supports exporting datasets and saved views to CSV and Excel.
- IO-12: V1 supports exporting views and dashboards to PDF.
- IO-13: A complete native workbench export is usable independently of cloud synchronization.

### Tentative

- JSON export.
- Image export.

### Proposed

- Import conflict options: keep both and rename, replace, or skip.
- Explain the effect of replacement on connected content.

### Open

- Excel feature compatibility and formula translation.
- PDF layouts and supported content.
- Native file contents, versioning, and attachment packaging.
- Native file behavior relative to view-only sharing snapshots.
- JSON import.
- Resolution of identity and dependency conflicts during import.

## 14. Notes and Comments

### Confirmed

- RECORD-01: Records support notes and comments.
- RECORD-02: Authorized users can comment on shared records.
- RECORD-03: Comments identify their author and creation time.
- RECORD-04: Threaded replies are a desired capability; release timing remains undecided.
- RECORD-05: @mentions are a desired capability; release timing remains undecided.
- RECORD-06: V1 records support rich-text notes.
- RECORD-07: V1 record details display and allow editing of all fields, including fields hidden in the current view.

### Proposed

- Begin with a chronological comment list before adding richer discussion features.

### Open

- Mention notifications.
- Comment editing, deletion, and permissions.
- Exact rich-text formatting and editing capabilities.

## 15. Release Scope and Quality Requirements

### V1 Definition

V1 is a complete personal data-management application with cross-device synchronization. Users must be able to create a workbench, enter or import data, connect datasets, calculate values, build saved views and dashboards, configure basic automations, and access synchronized workbenches on another computer.

V1 includes the confirmed requirements in Sections 1–8 and 11–14 except where a requirement is explicitly identified as post-V1 or remains an unresolved design detail.

### Confirmed V1 Scope

- Standalone Electron desktop application, built, packaged, and tested for Windows, macOS, and Linux.
- Independent workbenches, nested folders, flexible explorer, tabs, split panes, and explicitly opened additional windows.
- Full everyday spreadsheet editing and configurable validation.
- All confirmed baseline field types, connected references, and expanded formulas.
- Spreadsheet and configurable card views.
- Interactive dashboards with shared filters and record click-through.
- Managed attachments, rich-text record notes, and global search.
- CSV and Excel import; CSV, Excel, PDF, and native export.
- Local-first operation, optional accounts, and personal cross-device synchronization with conflict recovery.
- Basic local automations with configurable missed-run behavior.
- Autosave, persistent undo/redo, and configurable trash.
- Comprehensive accessibility and personalized appearance.
- Built-in application updates with safe data migration.

### Post-V1 Capabilities

- Multi-user sharing, permissions, and collaborative editing.
- Collaborative comments, threaded replies, and @mentions.
- Recovery snapshots and arbitrary earlier-state restoration.
- A mobile companion application.
- Always-on server-side automation execution, external integrations, and complex workflow orchestration.
- Additional view types such as Kanban boards and a freeform card-layout designer.

### Quality Requirements

- V1 targets smooth everyday editing, filtering, sorting, and searching with tens of thousands of records per dataset.
- Search and core data operations must work offline against locally available data.
- Locally saved changes must survive application restarts.
- Interrupted synchronization must not silently discard changes.
- Updates and migrations must preserve local data.
- Accessibility testing is part of release readiness.

### Remaining Decisions Before Implementation

- Minimum supported operating-system versions and installers.
- Measurable performance and responsiveness benchmarks.
- Local storage, synchronization, and account architecture.
- Reliable edit ordering and detailed conflict-resolution rules.
- Attachment limits, quotas, and offline availability.
- Exact formula syntax and function catalog.
- Automation execution safeguards, retries, and catch-up limits.
- Initial chart catalog and detailed dashboard behavior.
- Import mapping, compatibility, and identity-conflict handling.
- Detailed durability and accessibility acceptance criteria.

## 16. Validation Workbenches

- Food Operations is the primary validation workbench, covering inventory, recipes, and food-business data.
- Gaming is the secondary validation workbench, covering collections, quests, achievements, and progress.
- Both use general Workbench capabilities and remain independent, helping validate reuse within a workbench and isolation between workbenches.
- Detailed test workflows remain to be defined.
