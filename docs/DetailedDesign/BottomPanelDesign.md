# Workbench V1 — Bottom Panel Detailed Design

**Status:** Approved detailed design — 12 decisions locked (2026-10-19)  
**Target repository path:** `docs/DetailedDesign/BottomPanelDesign.md`  
**Scope:** Problems, Operations, History, Bottom Panel navigation and presentation, filtering, actions, Status Bar integration, and cross-surface behavior.  
**Companions:** `../WorkbenchV1Specs.md`, `../SystemDesign.md`, `../UIUXDecisionSet.md`, `ShellDesign.md`, `LeftPanelDesign.md`, `WorkAreaDesign.md`, `RightPanelDesign.md`.

## 1. Purpose, authority, and boundaries

The Bottom Panel is Workbench's compact, docked, cross-resource surface for diagnostics, ongoing/background operations, and canonical change History. It is an inspection and action surface, not an alternative resource editor or independent authority for data mutations.

- The V1 product specification, system design, and UI/UX Decision Set govern product semantics and architecture. This document refines presentation and interaction without adding or removing product functionality.
- Explicitly approved Bottom Panel decisions BP-01–BP-12 govern this design. Later explicit approvals supersede conflicting earlier presentation choices; unresolved conflicts require clarification rather than silent reinterpretation.
- Workbench Core owns canonical data, validation, durable problem/operation/history information as applicable, transactional writes, execution, permission checks, and supported Undo/Redo. The UI owns panel visibility, focus, selection, expansion, grouping, filtering, and other presentation state.
- Stable UUIDs and appropriate Core-issued identifiers, not labels or list positions, link entries to resources, operations, revisions, and navigation targets.
- Legacy [`better-spreadsheet`](https://github.com/kyliejurgs/better-spreadsheet) is historical design context only and does not override approved Workbench behavior.
- Exact dimensions, typography, color tokens, icon choices, animation, and numerical responsive breakpoints remain deferred to progressive visual specifications. Illustrations are conceptual, not pixel specifications.

## 2. Approved decision register

| ID    | Area                 | Locked outcome                                                                                                                                                                               |
| ----- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BP-01 | Header and tabs      | Compact IDE-style header with Problems, Operations, History tabs; active tab clearly identified.                                                                                             |
| BP-02 | Organization         | Each tab independently switches between flat and resource-grouped lists and remembers its choice.                                                                                            |
| BP-03 | Detail inspection    | Entries expand inline; extensive details may open an appropriate Work Area tab without losing Bottom Panel context.                                                                          |
| BP-04 | Filters              | Compact on-demand filter control; visible active-filter indication; independently remembered filters per tab.                                                                                |
| BP-05 | Attention and reveal | Passive events update notifications/Status Bar without opening the panel; explicit navigation reveals the requested tab; immediate input uses inline/focused dialog handling.                |
| BP-06 | Density/layout       | Always-compact single-column scrolling list, regardless of panel dimensions; no adaptive side-by-side detail pane.                                                                           |
| BP-07 | Entry navigation     | Single-click selects; double-click navigates to source; explicit Go to Source available; safe tab reuse, drafts, and missing-target handling.                                                |
| BP-08 | Counts               | VS Code-style single count badges beside relevant tab labels; separate Problems and Operations indicators in persistent Status Bar; counts independent of filters; no default History count. |
| BP-09 | Scope                | Workbench-wide by default; optional Active Resource or Specific Resource scope; per-tab remembered scope; badges and Status Bar remain Workbench-wide.                                       |
| BP-10 | Header utilities     | Filter, maximize/restore, close; reserve ability to add overflow when justified by actual secondary commands.                                                                                |
| BP-11 | Row presentation     | Dense, single-line rows with status/severity icon, primary message, muted resource context, and inline expansion; consistent at all heights.                                                 |
| BP-12 | Row actions          | Applicable contextual quick actions on hover/keyboard focus, with equivalent labeled actions in expanded details; action activation does not inadvertently select, expand, or navigate.      |

## 3. Placement, opening, and geometry — BP-01, BP-05, BP-06, BP-10

### 3.1 Docking and initial state

- The Bottom Panel sits **below the Work Area only**, not beneath the Left or Right Panels. The persistent Status Bar remains below all docked content.
- The default shell starts with the Bottom Panel hidden. The approved **Analysis** layout preset may reveal it; other presets follow `ShellDesign.md`.
- Opening or closing the Bottom Panel does not close editors, cancel operations, discard drafts, alter canonical resources, or change the current editor group.
- Panel height is user-resizable. Preserve the user's preferred height independently of temporary effective constraints, following Shell geometry rules. When space is limited, preserve a viable Work Area rather than treating the constrained height as a new preference.
- The Bottom Panel uses a **single-column scrolling list** at every size. Extra height exposes more rows, not a two-pane list/details layout.

### 3.2 Intent-aware reveal

- Passive validation events, background operation changes, and new History entries **do not automatically reopen** a hidden Bottom Panel or switch its active tab. They update the relevant indicators and applicable restrained notifications.
- Explicit commands such as **Show Problems**, **Show Operations**, **View Problem**, **View Operation**, or selecting the corresponding Status Bar indicator reveal the panel and select the intended tab. When a specific entry is targeted, locate it, reconcile filters as needed without silently destroying saved filter choices, and move focus meaningfully.
- Situations requiring immediate input are handled in the appropriate editor/inline surface or focused dialog, not by surprise Bottom Panel expansion.
- Ordinary editor selection or context changes do not override a deliberate user choice to hide the panel. Responsive constraints and explicit-reveal priority follow `ShellDesign.md`.

### 3.3 Maximize and restore

- The compact header provides **Filter**, **Maximize/Restore**, and **Close** controls.
- Maximize temporarily gives the Bottom Panel the available central Work Area region, subject to minimum viable shell constraints. It does **not** cover the Left Panel, Right Panel, title/menu area, or Status Bar.
- Restore returns to the previous user-preferred panel height. Maximization is a transient presentation state, not a destructive resize or a new preferred height.
- The header can gain an overflow (`…`) control later if real secondary commands justify it. No empty or speculative menu is required in V1.

## 4. Tab strip and indicators — BP-01, BP-08

The header exposes three persistent tab destinations: **Problems**, **Operations**, and **History**. Switching tabs changes only the Bottom Panel content, not the active Work Area editor. Each tab maintains its own grouping, filters, scope, and appropriate local selection/scroll context.

- Active tab is clearly indicated through more than color alone.
- **Problems** shows a single compact badge with the count of currently relevant unresolved Problems across the Workbench. The badge is **not** split into multiple numbers.
- **Operations** shows a single compact badge reflecting operations that are currently running or require attention; completed historical operations do not permanently inflate an attention badge. Define exact inclusion consistently with Core operation state and Status Bar semantics.
- **History** has no default count badge.
- Hide or de-emphasize zero-value badges consistently with the final visual system; do not imply an alert when there is none.
- Badges are based on actual Workbench-wide state, **not** the filtered or resource-scoped visible row count.

### 4.1 Persistent Status Bar

The Status Bar remains visible when the Bottom Panel is hidden. It separately exposes **error and warning counts** and relevant **running/attention-required operation indicators**, using icons and accessible labels. Selecting a Problems or Operations indicator explicitly reveals the matching Bottom Panel tab. These indicators derive from the same authoritative state as the tab badges, but need not use the same presentation format. Do not let a filtered list imply that Workbench-wide issues have disappeared. Preserve existing Status Bar responsibilities for active resource, freshness, and other Core state.

Conceptual presentation:

```text
Bottom Panel:  PROBLEMS [5]   OPERATIONS [2]   HISTORY       Filter  Maximize  Close
...
Status Bar:   Errors 3  Warnings 2  |  Running 1  Failed 1  |  other status
```

Numbers are illustrative. The tab badges are **single totals**, VS Code-style; the Status Bar supplies the category breakdown.

## 5. Organization, scope, and filtering — BP-02, BP-04, BP-09

### 5.1 Flat and grouped views

- Each tab supports **Flat** and **Grouped by Resource** presentation, independently remembered.
- Flat view displays a coherent tab-appropriate list. Grouped view adds collapsible resource headings; grouping does not alter entry identity, lifecycle, ownership, canonical ordering semantics, or resource placement.
- Group labels must disambiguate same-named resources. Entries without a usable resource association remain visible in an appropriate Workbench/global or unavailable-source group rather than disappearing.
- Group collapse and row expansion are presentation-only states. A change of grouping should preserve a meaningful selection where possible.

### 5.2 Resource scope

- New/default tab scope is **All Workbench Resources**.
- **Active Resource** follows the currently active Work Area resource as the user changes editors. Where an editor is not backed by a scoped resource, display a clear no-active-resource state rather than silently selecting a different scope.
- **Specific Resource** is pinned to an explicitly chosen resource and does not follow editor changes. If that resource is moved, renamed, trashed, or unavailable, retain the stable identity and explain the state; do not redirect by matching its display name.
- Scope is remembered **independently for Problems, Operations, and History** and appears in the existing compact filtering controls.
- Workbench-wide badges and Status Bar indicators remain global even when a tab shows a subset.
- Core-owned operations and History entries may involve multiple resources or the Workbench as a whole. Scope matching must use meaningful associations rather than arbitrarily assigning every entry to a single resource; global entries must remain discoverable under All.

### 5.3 Compact filter control

- Filtering/search controls are **on demand**, opened by the header filter button; no permanent second toolbar is required.
- A visible active-filter indicator remains when filters are applied and the controls are closed. Scope, search, and other applicable predicates are presented clearly, with a discoverable reset/clear path.
- Each tab remembers its own search, filter, and resource-scope settings; switching tabs does not overwrite another tab's configuration.
- Filters must be meaningful to each tab: e.g., severity/state for Problems, execution status for Operations, and relevant History metadata. Do not fabricate unsupported Core query fields.
- A no-match result is distinguished from a genuinely empty list, and offers a clear way to relax filters.
- An explicit navigation to an entry hidden by filters must explain/reconcile the filter conflict without silently replacing a user's remembered preferences.

## 6. Shared entry behavior — BP-03, BP-07, BP-11, BP-12

### 6.1 Dense rows and selection

Each row is a compact **single-line** entry with status/severity icon, primary message, and muted resource context. Long content is truncated legibly with accessible full-text disclosure. The list remains single-column at all heights; do not automatically switch to two-line or split-detail layouts.

- **Single-click** selects an entry without navigating away from the active editor.
- **Double-click** invokes source navigation when a valid source exists.
- **Inline expansion** exposes fuller message, structured context, available actions, and diagnostic details. Expansion is independent of the current Work Area editor.
- Selection, expansion, and scroll state should survive ordinary tab switching and return when feasible; if the entry is resolved, removed, or filtered out, provide sensible state restoration without selecting an unrelated entry.
- Keyboard and touch equivalents must be available; essential information and actions cannot require a hover-capable pointer.

### 6.2 Source navigation

- Expanded details expose an explicit **Go to Source** action where a navigable source exists.
- Navigate using stable IDs to the appropriate resource/editor and affected Record, Field, Query location, or other supported target. Reuse an existing Work Area tab where appropriate, respecting preview/permanent/pinned behavior and the two-group ceiling.
- Preserve unsaved/recoverable drafts and existing editor-local state. Reveal excluded Records through approved temporary View overrides, **not** by silently rewriting saved View filters.
- If a source is trashed, missing, moved, or inaccessible, keep the entry inspectable and show a meaningful unavailable-source or repair/recovery path. Do not navigate to a different resource with the same name.
- Navigation does not implicitly change a pinned Right Panel inspector; ordinary contextual inspectors follow the approved Right Panel rules.

### 6.3 Quick actions

- On row **hover or keyboard focus**, show only one or two contextually relevant quick actions, when available. Expanded details provide equivalent **labeled** actions and any additional applicable actions.
- Actions may include **Retry**, **Cancel**, **Go to Source**, **View Details**, or supported repair/recovery commands, depending on entry type and current Core state. An unavailable capability must be absent or appropriately disabled with explanation, never presented as guaranteed.
- Clicking an action does **not** accidentally trigger row selection, expansion, or double-click source navigation. A command runs once through the shared command/Core authorization path.
- Potentially destructive or high-impact actions follow the existing confirmation/impact-preview requirements. Failed actions preserve the diagnostic entry and actionable context.
- Context menus and keyboard command invocation follow shared Workbench command and accessibility conventions.

### 6.4 Optional Work Area details

Complex diagnostics, operation reports, or History comparisons may open a **full detail view in a Work Area tab**. This is optional for extensive content, not a replacement for ordinary inline inspection. Use existing tab identity/reuse and safe-draft rules. Opening the detail tab preserves the Bottom Panel's selected entry, filters, grouping, and expansion so users can return without losing context.

## 7. Problems tab

Problems presents currently relevant issues surfaced by approved validation, dependency, recovery, and integrity workflows. Examples include:

- Validation errors and warnings, with severity and affected target.
- Broken resource/Field/View/Query references and other actionable dependency issues.
- Save/commit conflicts, invalid or recoverable drafts, and applicable repair states.
- Relevant operation diagnostics where Problems is the approved persistent issue destination, without confusing the Problems list with the Operations execution log.

### 7.1 State and action semantics

- **Errors** block the affected invalid commit as defined by the Core; **warnings** permit commits where the product specification allows. Do not infer that every warning is a failed write.
- Distinguish unresolved, resolved, stale, and temporarily unavailable diagnostics as supported by the authoritative model. A problem must not disappear solely because a user changed tabs, filters, or active resource.
- Present message, severity, origin, actionable context, and relevant repair or navigation options. Use the appropriate existing editor, Right Panel, or focused workflow for remediation.
- For invalid drafts and conflicts, preserve the draft and expose saved-versus-proposed state and recovery paths as specified elsewhere; do not silently overwrite the canonical version.
- For resolved or no-longer-relevant issues, update badge counts from authoritative state, avoiding stale false alerts. If the system retains historical diagnostics, do not count them as unresolved merely because a log record remains.

## 8. Operations tab

Operations exposes approved long-running and background work such as imports, exports, query execution/refresh, derived recalculation, and Automation runs when supported.

- Show a compact status row with meaningful operation label, associated resource(s), and state: e.g., queued, running, completed, failed, cancelled, or another Core-defined state.
- Expanded details can show progress, steps, timestamps, errors, reports, and permitted actions. Progress may be indeterminate when no trustworthy percentage exists; never invent a numeric estimate.
- **Cancel** is available only while the Core says the operation is cancellable; cancellation requests can be pending and are not represented as immediately complete.
- **Retry** is available only when supported and authorized; it represents a new validated attempt, not an assumption that replay is safe or idempotent.
- Closing the Bottom Panel, switching tabs, changing editor, or maximizing/restoring does not cancel a Core operation.
- Completed operations remain accessible under approved retention/history semantics but do not indefinitely inflate the attention count. Failed/attention-required operations remain visible and actionable.
- Distinguish canonical write success from derived-data freshness and ongoing computation. A saved resource can coexist with stale, computing, or errored derived results.
- Automation activation, trust review, and scheduling rules remain those of the approved Core/Automation design; the Operations tab cannot bypass them.

## 9. History tab

History displays **Core-backed semantic changes** and supported Undo/Redo or comparison information, not arbitrary UI selection, panel visibility, or navigation events.

- Rows identify the change/action, affected resource(s), and relevant time/actor or revision metadata where supported. Use stable IDs and Core revision semantics, not display-name matching.
- Expanded details show available change context, affected objects, and applicable navigation/comparison/reversal actions.
- A complex diff or History comparison may open a dedicated Work Area detail tab, preserving Bottom Panel context.
- Undo/Redo availability and boundaries are Core-authoritative. Do not imply every History row is reversible or offer a generic rollback command for irreversible operations.
- A History action follows transactional/version-safe validation and conflict handling; it must not overwrite newer work or silently discard recoverable drafts.
- Global and multi-resource changes remain discoverable under Workbench-wide scope. Resource filtering is a view over supported associations, not a rewrite of canonical History.
- History has **no default count badge**; the tab remains a normal navigation destination.

## 10. Empty, loading, error, and recovery states

Provide distinct, compact, accessible states for each tab:

| State                        | Presentation and recovery                                                                                                           |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| No Problems                  | Quiet confirmation that no currently relevant problems are reported; do not claim all data is valid if diagnostics are unavailable. |
| No Operations                | Explain that no operations match the current view or are recorded, as appropriate.                                                  |
| No History                   | Explain that no applicable Core-backed changes are available.                                                                       |
| Filtered no matches          | Indicate that entries may exist outside the selected search/filter/scope; offer Clear/adjust filters.                               |
| Loading or refreshing        | Preserve useful prior content where safe, identify in-progress refresh, and avoid misleading zero counts.                           |
| Diagnostic/service error     | Show the unavailable data source and actionable retry/recovery; distinguish retrieval failure from absence of problems.             |
| Missing resource/source      | Retain diagnostic context and stable identity; offer repair, relink, restore, or inspection when supported.                         |
| Operation interruption       | Surface confirmed state and uncertainty; do not infer completion from UI disconnection.                                             |
| Conflicting or invalid draft | Preserve recovery information and explain next steps; never silently discard proposed edits.                                        |

Large lists should remain responsive through bounded retrieval, incremental rendering, and virtualization where appropriate. Selection, focus, grouping, and scroll restoration should use stable entry identity. Respect privacy and security boundaries for logs, parameters, File paths, and operation reports.

## 11. Accessibility and keyboard interaction

- Meet approved WCAG 2.2 AA goals where applicable, including visible focus, sufficient contrast, non-color status cues, and accessible names for icon-only controls and count badges.
- Support keyboard navigation between tabs, filter controls, grouped headings, rows, expanded details, contextual actions, maximize/restore, and close.
- Provide a keyboard equivalent for double-click source navigation and for hover-only quick actions. The exact keybinding must follow the shared command/keyboard design rather than introducing an inconsistent shortcut.
- Announce meaningful new errors, operation failures, and state changes through appropriate restrained accessible status mechanisms, without repeatedly stealing focus or reopening the panel.
- When an explicit command reveals the panel, move focus to the intended tab, entry, or action. Passive updates do not steal focus.
- Expanded/collapsed state, active tab, counts, severity, operation state, and current scope must be programmatically discernible.
- Resizing and maximize/restore require discoverable keyboard access; reduced-motion settings apply to any transitions.

## 12. Cross-panel integration and ownership

| Concern                                                                               | Contract / owner                                                                                                                                 |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Docking, layout presets, preferred geometry, intent-aware reveal, responsive priority | `ShellDesign.md`; Bottom Panel implements within those shell rules.                                                                              |
| Explorer and Search                                                                   | `LeftPanelDesign.md`; Bottom Panel source navigation can reveal a resource without changing Explorer's independent Search activity unexpectedly. |
| Work Area tab reuse, group focus, draft-safe navigation, temporary View overrides     | `WorkAreaDesign.md` and relevant resource editors.                                                                                               |
| Contextual Record/Field inspection, Notes, pinned inspectors, draft recovery          | `RightPanelDesign.md`; Bottom Panel does not override pins or silently discard drafts.                                                           |
| Validation, operation execution, authoritative History, Undo/Redo, persistence        | `SystemDesign.md`, `WorkbenchV1Specs.md`, and Workbench Core.                                                                                    |
| Status Bar badges and navigation                                                      | Shell Status Bar and shared state/command integration; indicators remain Workbench-wide.                                                         |
| Notifications and dialogs                                                             | Approved shell/feedback patterns; passive updates never force Bottom Panel reveal.                                                               |

### 12.1 State and restoration

Remember each tab's selected presentation mode, filter/search state, resource scope, and suitable session-local navigation context independently. Preserve the panel's requested visibility, preferred height, and safe session restoration in accordance with shell persistence. Temporary effective geometry, hover state, and other ephemeral interaction details must not be mistaken for canonical Workbench resource data. On reopen or restoration, reconcile unavailable entries and changed resources by stable identity and display recovery states rather than silently selecting a namesake.

### 12.2 Command and data boundaries

All row actions and Status Bar navigation use the shared command registry and consistent authorization, enablement, and focus semantics. The Bottom Panel may request operations or navigate to editors but does not write directly to canonical resources, bypass Core validation, forge History, or reinterpret a failed asynchronous operation as successful. Core updates may arrive asynchronously; stale/fresh states and version-safe updates remain explicit.

## 13. Final review and deferred visual specification

All **12 of 12 Bottom Panel decisions are approved**. The design intentionally leaves exact pixel heights, badge dimensions, typography, iconography, colors, motion, breakpoints, and the eventual need for a header overflow menu to later visual/implementation review. These are **not** new product decisions and must not be silently invented as locked requirements.

The Bottom Panel is complete at the detailed-design interaction level and is consistent with the approved compact IDE shell, dense Explorer and Work Area conventions, contextual Right Panel, Core-backed operation/History semantics, and persistent Status Bar. No GitHub changes are implied by this document.
