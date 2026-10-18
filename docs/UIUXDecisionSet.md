# Workbench V1 — UI/UX Decision Set

**Status:** Approved requirements baseline — 12 of 12 systems complete  
**Date:** 2026-10-17 
**Companion documents:** `docs/WorkbenchV1Specs.md`, `docs/SystemDesign.md`  
**Scope:** Product interaction, screen organization, visual design foundations, accessibility, and screen-design deliverables.

## Interpretation and precedence

This document consolidates the approved UI/UX decisions from the 12-system design-planning process. It supplements, rather than replaces, the approved V1 product and architecture baselines. The Core remains authoritative for domain data and operations; the React UI owns presentation, local editor state, and interaction. Where an older document conflicts with an explicitly later-approved decision, flag the discrepancy for reconciliation rather than silently discarding either requirement. In particular, the later-approved **one optional Markdown Note per supported parent** replaces older wording suggesting multiple Notes per target.

**Preserved architecture:** local-first desktop app; React + TypeScript and MUI; MōBrowser preferred, Electron fallback; authoritative versioned SQLite `.wkbn` file; one writable Workbench Core authority per open Workbench shared across windows; transactional canonical writes; asynchronous, version-safe derived calculations with explicit freshness; UUID resource identity; no background Automation execution when the relevant Core is closed. Resource transfers use valid native `.wkbn` databases, not a second package format.

**Approval convention:** Decisions below are locked functional/design-planning baselines. Screen-family designs will be provisionally approved and periodically reviewed together; refinements to presentation must preserve approved functional behavior. Exact visual tokens, accent palette, and polished screen compositions remain intentionally deferred to mockup review.

---

## System 01 — Application Shell & Layout (approved)

1. **Shell concept.** Spreadsheet-forward Data IDE: familiar data editing within a configurable professional desktop shell.
2. **Shell regions.** Title/menu area, integrated Command Center, optional Command Bar, Activity Bar, Left Explorer, tabbed Work Area, contextual Right Panel, Bottom Panel, and Status Bar.
3. **Docking and resizing.** Left, Right, and Bottom panels can show/hide, resize, and interact predictably with the Work Area; side panels use dock/push behavior, not accidental editor overlays.
4. **Panel constraints.** Minimum editor dimensions, resize limits, and responsive constraint handling; optional auto-collapse under pressure does not overwrite saved user preferences.
5. **Activity Bar.** Reorder, show/hide eligible entries, reset; activity selection toggles the corresponding panel. Keep essential navigation discoverable.
6. **Command surfaces.** Integrated title-area Command Center with optional separate command toolbar; consistent entry to search, navigation, and actions.
7. **Left/Right contextual roles.** Explorer/navigation on the left; contextual inspection and editing on the right, with user-controlled visibility.
8. **Bottom Panel.** Shared Problems, Operations, and History surfaces, collapsible and resizable.
9. **Status Bar.** Persistent access to relevant Core state, operations, Problems, active resource, freshness, and contextual counts; critical signals cannot be hidden.
10. **Layout persistence.** Per-Workbench preferred layouts and global defaults; distinguish preferred geometry from effective constrained geometry.
11. **Layout presets.** Default, Focus, Inspect, and Analysis, with user adjustments supported.
12. **Window behavior.** One Workbench or Home per window; multiple windows can share one Workbench Core, with independent UI layout and focus.
13. **Opening Workbenches.** Reuse/activate an already open Workbench authority rather than starting another writer; new Workbenches normally open in a new window.
14. **Session restoration.** Restore previous windows, tabs, editor groups, panels, and layout by default, subject to safe recovery.
15. **Panel headers and controls.** Predictable titles, contextual actions, close/collapse controls, and resize affordance.
16. **Two-group editor ceiling.** At most two editor groups, with horizontal or vertical split; independent group state.
17. **Focus and presentation.** Provide focused layouts without destroying saved panel preferences.
18. **Default shell.** Left Explorer open; Right and Bottom panels closed; one editor group; optional Command Bar hidden.

## System 02 — Navigation & Explorer (approved)

1. **Unified Explorer.** Navigate Workbench resources through one tree with clear root and Collection grouping.
2. **Hierarchy.** Collections are flat/non-nestable organizational containers; Tables, Queries, Dashboards, and Automations can be at root or in a Collection; Files are Workbench-root resources.
3. **Collection semantics.** Collections remain non-nestable organizational containers. Deleting a Collection moves the Collection and all resources currently contained within it to Trash as one recoverable cascading operation. Resources retain their UUIDs and ownership relationships. Restoring the Collection restores its contained resources and placement, subject to dependency and conflict validation.
4. **Table expansion.** Tables expand to Views only; Fields are managed in contextual Right Panel and Schema Manager.
5. **Selection/opening.** Single-click selects; double-click opens permanently; support multi-selection and keyboard equivalents.
6. **Creation.** Unified New entry points with context-sensitive resource choices and valid destinations.
7. **Context actions.** Consistent context menus and bulk actions backed by shared commands and Core validation.
8. **Drag and drop.** Move eligible resources between root and Collections; Files remain root-only; Views remain Table-owned.
9. **Explorer ordering and search separation.** Explorer supports Name, Type, Modified, Opened, and Manual ordering where applicable, respecting the resource hierarchy. Sorting controls are available through the Explorer header overflow menu. Explorer does not provide a dedicated name filter, search field, or resource-filtering mode. Resource and content discovery belong to the dedicated Left Search panel; rapid navigation belongs to the Command Center/Quick Open. Explorer remains focused on browsing and organizing resources.
10. **Quick Open versus Search.** Quick Open navigates rapidly; persistent Left Search offers broader discovery and results.
11. **Command Center modes.** Plain text for Quick Open; `>` commands; `@` Records; `#` resource types.
12. **History/performance.** Per-window Back/Forward and recent destinations; virtualize large Explorer trees.

## System 03 — Tabs & Work Area (approved)

1. **Tab classes.** Preview, permanent, and pinned tabs with distinct behavior.
2. **Tab identity.** One tab per View per editor group; a Table resolves to its last-used valid View or required default View.
3. **Cross-group behavior.** The same View may be open in both editor groups without duplicating canonical data.
4. **Tab strip.** Compressed tabs, scrolling/overflow, and clear active/pinned indicators.
5. **Editor groups.** Maximum two, split horizontally or vertically, with resizable divider.
6. **Tab movement.** Drag to reorder, move between groups, or open in a separate window where supported.
7. **Command context.** Focused editor/group determines contextual commands and panel content.
8. **State restoration.** Restore local selection, scroll, drafts, and other relevant editor-local state safely.
9. **Closing/reopening.** Safe close behavior, preserved or prompted drafts, independent ongoing Operations, and Reopen Closed Tab.

## System 04 — Grid Interaction (approved)

1. **Grid composition.** Spreadsheet-like row gutter, typed Field headers, View toolbar, and optional formula/value bar.
2. **Selection.** Rectangular ranges, multi-range selection, keyboard navigation, and coherent selection semantics.
3. **Editing.** Select versus edit distinction; F2/double-click edit; typing may replace selected content; Enter/Tab commit; Esc cancels.
4. **Click-away validation.** Valid edits commit; invalid drafts remain local and actionable rather than silently saving.
5. **Record creation.** Virtual blank rows create Records only on valid committed input.
6. **Clipboard.** Rich internal typed clipboard plus TSV/HTML interoperability, Paste Special, and precommit validation.
7. **Fill.** Familiar fill handle with typed, validated behavior.
8. **Bulk editing.** Find, Replace, and Transform with preview and Core-validated atomic or explicitly accepted valid-subset semantics.
9. **Typed editors.** Appropriate inline or popover editors for each Field type.
10. **Field actions.** Header menus for sorting, filtering, configuration, formatting, and schema-related actions.
11. **View presentation.** View-specific widths, Field order, visibility, frozen Fields, and density.
12. **Record actions.** Row selection and context actions; clear access to Record Details.
13. **Contextual inspection.** Right Panel for Record, Field, and Note information and edits.
14. **Grid toolbar.** Context-aware, customizable eligible actions and overflow.
15. **Density.** Compact, Comfortable (default), and Spacious, saved per View.
16. **Large data.** Virtualized scrolling, bounded fetches, responsive rendering, and stable navigation.
17. **Feedback/states.** Inline errors, selection/focus clarity, empty/loading/error states, and relevant status indicators.
18. **Identity/concurrency.** UUID-backed selection and draft state; version-aware updates and safe conflicts.

## System 05 — Fields, Records & Views (approved)

1. **Field management.** Everyday Field operations in Right Panel; advanced structural work in Schema Manager; distinguish canonical Field order from View order.
2. **Field creation/conversion.** Guided type selection and conversions, with impact previews for invalid, lossy, and dependency-affecting changes.
3. **Record Details.** Contextual Right Panel quick editing, with expanded editor/tab for complex records.
4. **Record organization.** Searchable Record sections: Overview, All Fields, Relationships, Attachments, Notes, Activity; optional Field grouping.
5. **Relationships.** Linked Record picker and relationship browser, reverse links, and Preserve/Clear/Restrict deletion policies.
6. **View creation.** Guided creation for Grid, Kanban, Calendar, Gallery, and Pivot.
7. **View selector.** Favorites, reorder, rename, duplicate, set default, Trash, and protection for the required default View.
8. **Saved versus temporary.** Distinct saved View configuration and temporary overrides; Save to View, Reset, Save as New, and recoverable overrides; Record edits autosave.
9. **View switching.** Activate existing View tab or open it without duplicate tabs in a group.
10. **Configuration placement.** Toolbar popovers for frequent controls; Right Panel for advanced configuration.
11. **Grid-specific configuration.** Grouping, summaries, conditional formatting, layout, and density.
12. **Kanban.** Validated drag between lanes, editable cards, summaries, and unassigned lane where relevant.
13. **Calendar.** Month/Week/Day/Agenda, Field mapping, drag/resize with validation, unscheduled items, timezone behavior.
14. **Gallery.** Cover, title, metadata, card size, quick edit, and lazy rendering.
15. **Pivot.** Rows, Columns, Values, Filters; read-only aggregate cells and drill-through.
16. **Shared data semantics.** Type-aware filtering, sorting, grouping, and summaries across Views.
17. **Reveal Record.** Temporary View override to reveal a Record excluded by filters.
18. **Live synchronization.** Core-backed cross-View updates preserving focus and handling conflicts.
19. **Repair.** Recoverable broken View configurations with guided repair.
20. **Scale.** View-specific virtualization and Core-backed aggregation where needed.

## System 06 — Queries & Dashboards (approved)

1. **Query modes.** Visual and SQL are separate authoring modes; no automatic bidirectional editing; generated SQL inspectable; Duplicate as SQL.
2. **Visual Query Builder.** Source, joins, fields, filters, groups, sorting, and computed expressions through structured UI.
3. **SQL editor.** Code editor with completion and diagnostics; approved read-only SELECT boundary.
4. **Execution.** Manual Run by default, optional automatic execution, cancellation where safe, and preserved stale prior results.
5. **Query layout.** Resizable definition/results split; read-only virtualized typed results.
6. **Parameters.** Named typed parameters with defaults, safe binding, editor-local values, and mappings.
7. **Dependencies.** Right Panel dependency information and impact navigation.
8. **Freshness.** Fresh/Stale/Computing/Error with version-safe recalculation and explicit previous results.
9. **Execution history.** Bounded, searchable metadata and privacy-conscious parameter handling.
10. **Result reuse.** Query sources for Dashboards and other Queries, cycle checks, export, drill-through, Automation integration, and explicit Materialize Results to Table snapshot.
11. **Dashboard modes.** Explicit View/Edit; staged layout changes with Save and recoverable drafts.
12. **Dashboard layout.** Responsive 12-column snap grid, drag/resize, collision handling, keyboard operation, canonical nonoverlap.
13. **Widget types.** KPI, chart, Table, Pivot, Text, Filter, Record List, and Media, owned by Dashboard.
14. **Data sources.** Unified Table/View/Query source picker with UUID mapping, freshness, and repair.
15. **Filters.** Global and widget filters, temporary cross-filter, drill-through, saved defaults distinct from transient interaction.
16. **Widget commands.** Configure, Duplicate, Refresh, Inspect Data, Remove, with contextual Right Panel.
17. **Widget freshness.** Per-widget and aggregate dashboard freshness.
18. **Drill-through.** Context-preserving UUID navigation and temporary destination filters.
19. **Responsive presentation.** Reflow and Focus/Presentation mode.
20. **Widget states.** Explicit empty, loading, error, and invalid-configuration repair states.

## System 07 — Files, Notes & Automations (approved)

1. **Files navigation.** Root Files section with type/storage/usage grouping, filtering, sorting; grouping is presentation only.
2. **Storage mode.** Explicit embedded versus external File choice, status badges, relink, and conversion.
3. **File editor/preview.** Markdown live/source, text/JSON/CSV, images/PDF previews, supported Office handling or external fallback; Core-backed edits and drafts.
4. **Files and attachments.** Reusable picker to attach existing or add new File. Files remain Workbench-root resources with stable UUIDs. Independent Files can exist without attachments. Attachment associations reference existing Files without duplicating content, and a File may be associated with multiple resources. Ordinary detachment does not delete the File. When deleting a resource, a **Preserve attachments** checkbox is shown unchecked by default in the deletion impact preview. By default, attachment-managed Files exclusively associated with the full deletion set are also moved to Trash. Checking Preserve attachments retains those Files independently. Files referenced by surviving resources and intentionally independent Files are preserved. Restore and Undo preserve applicable File identities and attachment associations.
5. **File operations/transfer.** Guided operations, `.wkbn` native transfers, explicit external-File inclusion choices and Include All; no second native package format.
6. **Note editor.** Markdown Live Preview/Source, shortcuts, autosave, drafts, expandable editor.
7. **Note ownership.** **One optional Markdown Note per supported parent**: Workbench, Collection, Table, Field, Record, View, Query, Dashboard, Automation, File. Parent-owned UUID lifecycle and uniqueness; no standalone Note lists, pinning, ordering, duplication, or reassignment. Headings, collapsible outline, and checklists are allowed. Markdown Files remain separate root Files.
8. **Note placement.** Contextual Right Panel Note with outline; optional full editor tab.
9.  **Note links.** Standard hyperlinks and UUID-based internal resource links with repair; links do not confer ownership.
10. **Note search/history.** Heading-aware full-text search, History/Undo, drafts, asynchronous indexing.
11. **Automation creation.** Guided templates or blank definition, resource pre-selection, disabled until valid/authorized; drafts and safe duplication.
12. **Automation builder.** Declarative vertical trigger/condition/action blocks, AND/OR, branches, mappings, contextual Right Panel; no scripts.
13. **Triggers.** Event, scheduled, and manual; timezone, recurrence, next run, Core-running-only schedules, missed-run policy, loop safeguards.
14. **Testing.** Dry-run impact preview without canonical mutations; separate manual Run; Operations/logs.
15. **Trust.** Imported/untrusted Automations disabled pending review; locally authorized Automations may resume; effect changes trigger re-review; Pause/Disable supported.
16. **Run history.** Bounded searchable detailed execution history.
17. **Status.** Configuration: Draft/Disabled/Enabled/Paused/Review Required/Invalid. Execution: Idle/Queued/Running/Succeeded/Failed/Cancelled/Skipped.
18. **Reliability.** Bounded safe retries, idempotence, partial-success reporting, Core transactions.
19. **Dependencies.** UUID impact/repair/freshness and cycle checks.
20. **Lifecycle.** Enable/Disable/Pause/Resume/Duplicate/Move/Trash/Restore; skip/report missed schedules by default, optional bounded catch-up; schedules run only while Core is active.

## System 08 — Commands & Contextual UI (approved)

1. **Registry.** Central command registry with stable IDs, context, enablement, validation, and execution.
2. **Palette.** Fuzzy categorized contextual Command Palette, recent/history, disabled reasons, shortcuts, aliases; Command Center modes `>`, `@`, `#`.
3. **Menus.** Shared command semantics for context menus, menus, and multi-select; consistent ordering.
4. **Shortcuts.** Customizable platform-aware shortcuts, conflict detection, Grid/focus handling.
5. **Targeting.** Focus-aware targets; Core revalidates permissions/context; confirmations, Undo/Redo, Operations as appropriate.
6. **Toolbars.** Eligible actions can be shown/hidden/reordered; overflow; mandatory critical actions preserved; UI preferences separate from canonical data.
7. **Assistance.** Progressive suggestions, tooltips, hints, and reference information; dismissible.
8. **Guided flows.** Preview/validation/drafts/Apply/Operations/version-aware multi-step workflows.
9. **Placement.** Predictable use of toolbar, inline controls, Right Panel, context menu, Palette, menu, and dialog.
10. **Feedback.** No per-edit toast; layered feedback through inline, Problems, Operations, and freshness.
11. **Accessibility.** Keyboard/focus/screen-reader parity for commands and menus.
12. **Bulk actions.** Selection-aware scope, preview, atomic or explicitly accepted valid subset, partial reporting, Undo where supported.
13. **Undo/Redo.** Core-backed semantic History, transaction boundaries, irreversibility, local draft Undo.
14. **Preference scope.** Global command defaults and eligible per-Workbench overrides; shortcuts global; toolbar defaults with Workbench overrides; bounded global Palette history.
15. **Command contracts.** Stable metadata for ID, category, aliases, targets, permissions, trust, validation, cancellation, Undo, feedback, shortcut, visibility; extensions use registry.

## System 09 — Visual Design Foundations (approved, with deferred tokens)

1. **Direction.** Spreadsheet-forward Data IDE hybrid, professional and compact.
2. **Themes.** Light, Dark, Follow System (default); live switching.
3. **Palette reference.** Slate & Blue is provisional only; exact identity, default accent, and token values await full-screen mockup review.
4. **Accent.** Curated user-selectable palette plus custom color picker, contrast safeguards; accent on focus/selection/primary actions/active navigation, not semantic status or large backgrounds. Final swatches and scope are deferred.
5. **Typography.** Modern UI sans and technical monospace; Inter and JetBrains Mono are candidates, not locked fonts.
6. **Density.** Compact shell; Grid View-specific Compact/Comfortable/Spacious.
7. **Component styling.** Balanced modern styling and moderate rounding.
8. **Iconography.** Primarily outline icons, selective filled states; Lucide is a candidate.
9. **Depth.** Subtle borders and modest elevation, mostly on floating surfaces.
10. **Data surfaces.** Consistent styling across Grid, Query results, Pivot, and Dashboard widgets.
11. **Interaction states.** Distinct selection, focus, edit, hover, disabled, and drag feedback.
12. **Semantic indicators.** Restrained status color, escalating errors, separate saved versus derived freshness.
13. **Overlays.** Task-based hierarchy for popovers, dialogs, menus, and other overlays.
14. **Motion.** Subtle functional animation, reduced-motion support.

## System 10 — Settings & Accessibility (approved)

1. **Three-level Settings.** Quick Settings popover anchored to Settings Activity Bar icon; searchable Full Settings tab in Work Area; contextual Settings in Right Panel. All synchronized. Quick Settings has frequent controls and Open All Settings.
2. **Scope.** Global defaults with eligible explicit per-Workbench overrides; show inherited/overridden status and reset-to-global. Canonical View-specific settings remain separate.
3. **Persistence.** Ordinary settings apply and autosave immediately across windows; consequential changes require confirmation; restart requirements clearly labeled.
4. **Accessibility.** OS-aware defaults plus text scaling, reduced motion, stronger focus, contrast, keyboard aids, WCAG 2.2 AA target where applicable.
5. **Shortcuts editor.** Full searchable keyboard shortcut editor, conflicts, reset, platform-specific display, shared registry.
6. **Discovery.** Search labels, descriptions, categories, aliases; command deep links; Quick Settings need not duplicate full search.
7. **Reset.** Individual, category, and eligible application-wide reset, preview for broad reset; never delete Workbench data.
8. **Portability.** Explicit export/import of application preferences with preview; separate from `.wkbn` resource transfer/backups; sensitive/machine-specific values excluded or handled explicitly.
9. **Multi-window.** Live global and eligible per-Workbench preference synchronization; window-local layout, focus, tabs remain independent.
10. **Organization.** Task-oriented categories: General, Appearance, Workspace & Layout, Editor & Grid, Keyboard, Accessibility, Files & Storage, Backups & Recovery, Automations, Advanced, as applicable.
11. **Recovery.** Immediate reversible accessibility changes with keyboard-accessible reset/recovery; honor OS preferences unless overridden.

## System 11 — Feedback & Exceptional States (approved)

1. **Notifications.** Quiet routine edits; layered toasts for meaningful events; persistent issues in context/Problems, lengthy work in Operations.
2. **Errors.** Context-first inline actionable findings linked to centralized Problems; avoid duplicate unrelated alerts.
3. **Progress.** Non-blocking operation progress and persistent Operations history; queued/running/completed/failed/cancelled; safe cancellation only.
4. **Save/freshness.** Distinct canonical save and derived Fresh/Stale/Computing/Error indicators, with clear explanation.
5. **Messaging.** Plain-language what happened/impact/next action, expandable diagnostics.
6. **Conflicts.** Preserve local draft; show current saved value and explicit resolution; no silent overwrites or lost drafts.
7. **Drafts.** Recoverable invalid/interrupted drafts, revalidated before commit; never treat invalid draft as canonical.
8. **Dependencies.** Keep safe degraded resources accessible; identify missing dependencies and offer guided repair; do not silently remove configuration.
9. **Partial outcomes.** Item-level succeeded/rejected/skipped/failed summaries, correction and retry; respect atomic operations and explicitly accepted valid subsets.
10. **Crash recovery.** Guided reopen from last committed state, recover drafts, report interrupted operations and integrity/migration issues; no unsafe automatic rewriting.
11. **Confirmations.** Risk-based, with impact previews for consequential operations and explicit acknowledgment for irreversible actions; explain Table-owned cascades versus Collection cascade.
12. **Notification history.** Lightweight center with links to Problems, Operations, History, resources; noncritical preferences, critical safety notices preserved.
13. **Diagnostics.** Central diagnostics with contextual entry points, operation IDs/Core health/dependency status and privacy-conscious copyable details; exclude sensitive contents by default.
14. **Exceptional states.** Consistent empty, filtered-empty, loading, unavailable, error, invalid states with contextual recovery actions; preserve selection/drafts where safe.
15. **Accessible feedback.** Color-independent messages, keyboard actions, focus restoration, appropriate screen-reader announcements, restrained live regions.

## System 12 — Screen Workflows & Component Inventory (approved)

1. **Inventory.** Complete screen/panel/dialog/flyout/state inventory with purpose, entry points, commands, layout, dependencies, and requirements.
2. **Journeys.** End-to-end workflows including success, validation, cancellation, error, and recovery paths.
3. **Components.** Three-layer catalog: shared primitives, shared Workbench patterns, feature-specific components; use MUI for appropriate commodity controls.
4. **Fidelity.** Structural wireframes first, then polished full-screen mockups; decide final palette/tokens during visual review.
5. **Traceability.** Structured screen specs linked to requirements, components, keyboard, accessibility, responsive constraints, and states; separate unresolved visual choices.
6. **Sequence.** Shell/navigation first, then Grid/Records, then other Views, Queries, Dashboards, Files/Notes/Automations, and system surfaces.
7. **Variants.** Representative state matrix; reuse patterns rather than mock up every combinatorial state.
8. **Window coverage.** Compact, typical, and large desktop layouts; resizing, constrained space, two editor groups, and relevant focus modes; no new mobile app scope.
9. **Interaction contracts.** Pointer, keyboard, focus, drag/drop, commands, validation, persistence, accessibility linked to command registry and Core.
10. **Approval.** **Progressive approval with integration reviews:** screen-family approval establishes a provisional design baseline; periodically review shared patterns across families, revise intentionally, track affected screens. Do not reopen approved functional architecture by default.
11. **Screen specs.** One structured specification per screen family; shared shell and component specs separate.
12. **Design system.** Living shared typography, color, spacing, density, icons, borders, elevation, component/state catalog refined through mockups.
13. **Prototypes.** Representative interactive cross-screen workflow prototypes; not every screen requires a working prototype.
14. **Integration reviews.** Periodic cross-feature review of navigation, commands, panels, dialogs, feedback, keyboard, accessibility, with impact tracking.
15. **Completion.** Coverage-based: every approved function mapped to screen/interaction; major workflows, shared components, critical states covered; unresolved choices documented. No coded prototype/implementation validation required to call design planning complete.

---

## Cross-cutting product invariants to preserve

- **Data authority:** Canonical writes are transactional Core operations; derived updates are asynchronous, version-safe, and explicitly freshness-tracked. Dependent operations respect freshness guarantees.
- **Resource model:** UUIDs; non-nestable Collections; Collection deletion implicitly deletes contained resources; Table ownership of Fields/Records/Views has cascade behavior; Files root-only; one optional Note per supported parent.
- **Native format:** Live `.wkbn` SQLite database for Workbenches, copies, and resource transfers; external File inclusion is explicit with Include All.
- **View behavior:** Five native Views; saved View configuration distinct from temporary overrides and canonical Record edits; required default View.
- **Query safety:** Visual/SQL modes are separate; SQL restricted to approved read-only operations; materialization creates an explicit Table snapshot.
- **Automation safety:** Declarative only; imported/untrusted definitions disabled pending review; locally authorized may resume; scheduling only while Core runs.
- **Lifecycle:** Trash only, no separate Archive lifecycle; History/Undo/Redo, backups, and Trash are distinct mechanisms.
- **Accessibility:** Keyboard-only workflows, screen-reader context, visible focus, meaningful feedback, reduced motion, WCAG 2.2 AA target where applicable.
- **Performance:** Virtualization, bounded reads/results, responsive UI; previously approved approximate scale targets of 1M routine, 10M primary, 50M stress, 100M+ best effort remain planning goals, not promises of universal performance.

## Known documentation reconciliation

1. Older Workbench documentation may say multiple Notes per target. The later-approved UI/UX requirement is **one optional Markdown Note per supported parent**, with separate root Markdown Files.
2. This document describes UI/UX approvals not yet merged into the existing `WorkbenchV1Specs.md` or `SystemDesign.md`; reconcile references deliberately when updating documentation.
3. Better Spreadsheet's earlier application-layout/style documentation is a useful design precedent, **not** an architecture replacement. Do not carry forward its obsolete Angular/PWA/cloud synchronization assumptions.
4. The exact default accent, final palette, typography choices, dimensions, and visual tokens are **not locked**. They are to be decided with actual mockups.

## Next design phase

**Start Screen Family 01: Application Shell & Navigation.**

1. Produce a structural wireframe for the default Workbench desktop window based on the approved shell.
2. Cover Default, Focus/Grid, two-editor-group, and constrained-window variants.
3. Specify panel toggles, resizing, tabs, Explorer, Command Center, Right Panel, Bottom Panel, and Status Bar interactions.
4. Review shell/feature integration, then develop polished Light/Dark mockups and a living token system.
5. Provisionally approve the shell family, log changes and dependencies, and proceed to Grid/Record workflows.

**Repository action:** Add this Markdown document to the planned `ui-design` branch only when explicitly authorized. This file is a portable draft; generating it does not modify the repository.
