# Workbench V1 — Left Panel Detailed Design

**Status:** Approved detailed-design decisions (8 of 8); implementation-level visual tokens remain deferred  
**Date:** 2026-10-18  
**Target path:** `docs/DetailedDesign/LeftPanelDesign.md`  
**Scope:** Activity Bar, Explorer, Search, shared Left Panel interactions and states  
**Companions:** `../WorkbenchV1Specs.md`, `../SystemDesign.md`, `../UIUXDecisionSet.md`, `ShellDesign.md`, `WorkAreaDesign.md`, `RightPanelDesign.md`, `BottomPanelDesign.md`

## 1. Purpose, authority, and boundaries

This document specifies the Workbench V1 Left Panel without changing the approved product model or Core architecture. It records eight expressly approved Left Panel decisions and elaborates their necessary interaction contracts.

- `WorkbenchV1Specs.md` governs product semantics; `SystemDesign.md` governs architecture and Core authority; `UIUXDecisionSet.md` records the consolidated approved UI/UX requirements; `ShellDesign.md` governs the surrounding shell.
- A later expressly approved decision replaces conflicting earlier wording. Do not silently import legacy behavior where it conflicts with Workbench.
- The legacy [`better-spreadsheet`](https://github.com/kyliejurgs/better-spreadsheet) documents `docs/application-design/left-panel-views.md` and `docs/application-design/application-layout.md` inform presentation patterns only.
- This document does **not** redefine Work Area tab mechanics, canonical resource mutations, File management, search indexing, Right Panel editing, or shell geometry.
- Exact pixels, spacing scales, colors, icon set, animation curves, and responsive breakpoints are intentionally left to later visual specifications.

## 2. Approved Left Panel decision register

| ID    | Approved decision                | Result                                                                                                                                                                                             |
| ----- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LP-01 | Separate Explorer sections       | Workbench resources and Files appear in independently collapsible sections.                                                                                                                        |
| LP-02 | Compact Explorer rows            | Compact hierarchical rows with separate selection backgrounds and persistent active-resource indicators.                                                                                           |
| LP-03 | Quick action + overflow hybrid   | Relevant one-click quick action and consistent `…` menu appear on hover or keyboard focus.                                                                                                         |
| LP-04 | Automatic active-resource reveal | When the active Work Area resource changes, expand only required ancestors and scroll it into view; preserve other branches and closed-panel intent.                                               |
| LP-05 | Search grouping toggle           | User can choose grouped, collapsible categories or a unified relevance-ranked list; switching does not change query, filters, or results.                                                          |
| LP-06 | Activity Bar zones               | Reorderable/hideable eligible primary activities at top; stable application utilities at bottom.                                                                                                   |
| LP-07 | Inline contextual states         | Empty, loading, errors, and recovery appear within the affected section, leaving unaffected content usable.                                                                                        |
| LP-08 | Row click selects and expands    | Single-click on expandable row selects and toggles expansion; disclosure control toggles without selecting; double-click opens openable resources permanently; modifier selection does not toggle. |

These decisions refine, rather than reopen, the nine approved Shell Design decisions.

## 3. Region structure

The Activity Bar and Left Panel occupy their approved positions to the left of the Work Area. Only one Left Panel activity is displayed at a time; initial activities are **Explorer** and **Search**. The Left Panel uses the shell's saved preferred width, adaptive constraints, resize handle, visibility controls, and compact shared header. Activity-specific views do not independently resize the Left Panel.

```text
┌───────────┬──────────────────────────────────┬───────────────────────────┐
│ ACTIVITY  │ LEFT PANEL                       │ WORK AREA                 │
│ BAR       │ EXPLORER or SEARCH               │                           │
│           ├──────────────────────────────────┤                           │
│ Explorer  │ Compact title / actions / close  │                           │
│ Search    ├──────────────────────────────────┤                           │
│ …         │ Activity-specific scroll content │                           │
│           │                                  │                           │
│           │                                  │                           │
│ Utilities │                                  │                           │
└───────────┴──────────────────────────────────┴───────────────────────────┘
```

Selecting a different activity replaces the current Left Panel view, not the Work Area. Selecting the currently active Activity Bar item toggles Left Panel visibility. Selecting an activity while the panel is closed explicitly reopens it to that activity. The previously active activity, selection, scroll, tree expansion, search input, filters, results, and presentation preferences remain intact as appropriate.

The panel header uses the approved shell-wide pattern: title left; contextual actions, overflow, and close/collapse controls right. Header actions are keyboard reachable. Content below the fixed header scrolls independently.

## 4. Activity Bar

### 4.1 Organization and visuals

- **Top zone:** Explorer, Search, and any subsequently eligible primary activities.
- **Bottom zone:** Stable application utilities, including Settings and notification access; utilities do not have to activate a Left Panel activity.
- The active primary activity has a visible, color-independent active indicator and accessible selected state.
- Activity icons have tooltips and accessible names; keyboard focus is distinct from active state.
- Eligible top-zone activities can be reordered and hidden, with a reset mechanism. Essential navigation remains discoverable. Bottom utilities remain anchored and are not mixed into the primary activity ordering.
- Notification access is a utility entry point, not an automatically invented third Left Panel activity.

### 4.2 Interaction

- Inactive activity click/keyboard activation: activate that view and reveal Left Panel if closed.
- Active activity activation: toggle Left Panel visibility.
- Collapse/expand preserves the active activity and view-local state.
- Switching activities does not alter the active Work Area resource or close editor tabs.
- Shell commands can independently toggle the Left Panel; explicit navigation or panel-show commands may reopen it, while ordinary selection changes cannot override a deliberate collapse.
- Reordering/hiding follows accessible keyboard and pointer workflows; the shared command/settings model owns persistence and reset.

## 5. Explorer — information architecture

### 5.1 Sections

Explorer has **two independently collapsible sections**:

1. **Workbench:** The active Workbench name labels the resource section. The tree contains root-level Collections, Tables, Queries, Dashboards, and Automations. A Collection may contain Tables, Queries, Dashboards, and Automations. Tables expand **directly to their Views**. Collections cannot nest. No intermediate `Views` node is inserted.
2. **Files:** All Workbench-root Files, both independent and attachment-managed. Files never appear under Collections or Tables merely because they are referenced there.

```text
EXPLORER                                       +  …
▾ My Workbench
    ▾ Sales [Collection]
        ▾ Customers [Table]
            Grid [View]
            Calendar [View]
        Revenue Forecast [Query]
        Overview [Dashboard]
        Nightly Sync [Automation]
    Inventory [Table]
    Operations [Automation]
▾ FILES
    report.pdf
    logo.png
```

This is a presentation hierarchy, not a second ownership model. A Table owns its Views; a Collection organizes resources without owning their data. Fields, Records, Dashboard Widgets, and contextual Notes do not appear as standalone Explorer nodes. Fields belong in the Right Panel/Schema Manager; Notes are parent-associated.

### 5.2 Files section grouping — previously approved

The **default Files presentation is flat**. Users may choose virtual grouping by **Usage, Type, or Storage** from the Files section overflow menu. Group headings can collapse independently. This grouping does not change File ownership, UUIDs, attachment references, or placement. Files sorting is independent of grouping. Grouping and sorting preferences persist across restarts and reopening the Workbench; new Workbenches default to flat Files.

### 5.3 Section headers

Each section header has a disclosure control, section icon, label, and relevant contextual actions. Collapsing a section does not collapse the Left Panel. Section expansion is remembered. The Workbench section label uses the actual Workbench name; the Files section is labeled `FILES` or the design-system equivalent.

## 6. Explorer — row appearance and interaction

### 6.1 Compact row pattern

Rows use compact spacing, consistent type size, hierarchy indentation, resource-specific icons, and disclosure affordances on expandable resources. Text truncates rather than displacing necessary controls; full names remain accessible via tooltip or equivalent. The row's interactive target remains usable with keyboard and pointer.

- **Selected row(s):** subdued selection background.
- **Active Work Area resource:** persistent restrained accent indicator, independently of Explorer selection.
- **Selected and active:** both treatments are visible together.
- **Hovered:** transient hover treatment distinguishable from selected and active.
- **Focused:** explicit keyboard focus indicator, not color alone.
- **Disabled/unavailable:** reason discoverable; never silently suppress a consequential command.

Explorer selection is not the same as the active Work Area resource. Multi-selection may highlight several rows while one separately remains active in the editor.

### 6.2 Row activation — LP-08

| Gesture                                                         | Expandable Collection / Table                               | Nonexpandable openable resource      |
| --------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------ |
| Single-click main row                                           | Select and toggle expanded/collapsed                        | Select only                          |
| Click disclosure control                                        | Toggle expansion without changing selection                 | Not applicable                       |
| Double-click main row                                           | Open if openable (Table); Collection remains organizational | Open permanently                     |
| Ctrl/Cmd-click, Shift-click, or equivalent multi-select gesture | Change selection without toggling expansion                 | Change selection without opening     |
| Keyboard selection                                              | Change selection/focus independently                        | Change selection/focus independently |
| Keyboard Left/Right                                             | Collapse/expand using tree conventions                      | Navigate tree as applicable          |
| Keyboard open command                                           | Open applicable resource using shared command/tab semantics | Open applicable resource             |

**Double-click resolution:** Treat a recognized double-click on an openable row as the permanent-open gesture, not as two independent expansion toggles. Its final expanded/collapsed state should remain predictable and not oscillate. For a Collection, double-click must not fabricate an editor; the Collection remains an organizational selection. Pointer and keyboard interactions should provide equivalent functionality.

A Table opens through its last-used valid View, falling back to its required default View. Opening a View or other openable resource follows approved Work Area preview/permanent/pinned tab contracts; this document does not redefine them.

### 6.3 Selection and active-resource reveal — LP-04

When the active Work Area resource changes through tabs, Search, Quick Open, links, or another navigation source:

1. Resolve its UUID-backed Explorer node and ancestor path.
2. If Explorer is **visible**, expand only the collapsed ancestors required to expose that node and scroll it into view.
3. Apply the active indicator, independently of current Explorer selection.
4. Preserve unrelated expanded branches and any user multi-selection where practical.
5. If the Left Panel is hidden or Search is active, **do not forcibly switch activities or reopen Explorer**. The reveal may be applied when Explorer next becomes visible.
6. Do not repeat reveal or scroll jumps for ordinary cell/record selection changes, derived recalculation, or background updates that do not change the active Work Area resource.

Reveal must not rewrite a saved View filter or change canonical resource data. Record-level reveal in a View uses the approved temporary override/navigation contract, not Explorer tree mutation.

## 7. Explorer — commands and actions

### 7.1 Header `+` and contextual creation

Explorer's header `+` opens the shared New flow, with eligible types and destination defaulted from current selection:

- Collection selected, or selected resource belongs to a Collection: **that Collection** is the default destination for eligible root/Collection resources.
- Root-level resource selected, or no selection: **Workbench root** is the default.
- **Files:** root only, regardless of selected Collection.
- **Views:** owned by a Table; a selected Table or View may provide the owning Table as context when using a View-specific creation command.
- **Collections:** root only; never nested.
- Commands must honor Core validation, naming rules, dependencies, and resource-type eligibility.

The selected destination is a default, not permission to violate ownership rules. Creation must not silently move existing resources or invent new containment types.

### 7.2 Row quick actions and overflow — LP-03

On row hover **or keyboard focus**, show an eligible quick action and an overflow `…` entry:

- Collection: `+` can create an eligible resource in that Collection.
- Table: `+` can create a Table-owned View.
- Other resources: may show only `…` where no sufficiently common direct action is justified.
- Overflow opens the same shared-command-backed, context-validated action set as right-click.
- Action controls must not intercept normal row selection/expansion when activated; keyboard focus and accessible labels identify the exact target.
- Under narrow widths, prioritize legibility and overflow; do not truncate controls into unusable hit areas.

Context menus expose applicable Rename, Duplicate, Move, Export, Trash, and other approved commands, not a universal menu of invalid actions. Bulk commands operate on the selected set with explicit impact, validation, and partial-result semantics where supported.

### 7.3 Sorting and manual order

Explorer supports **Name, Type, Modified, Opened, and Manual** ordering where applicable, through the **Explorer header overflow**. Sorting respects the Workbench/Collection/Table hierarchy: it reorders eligible siblings, not ownership or ancestor paths. Manual ordering is available only where supported. The selected ordering is a presentation preference, not a UUID or ownership change.

Explorer has **no inline name search, name filter, or resource-filtering mode**. Use the dedicated Search activity for persistent discovery and Command Center/Quick Open for fast navigation.

### 7.4 Drag-and-drop and moves

- Eligible Tables, Queries, Dashboards, and Automations can move between root and Collections, subject to Core validation.
- Collections remain root-level and non-nestable.
- Files remain root-only; Views remain Table-owned.
- Show valid drop targets, rejected-target explanations, and clear keyboard alternatives.
- Drop feedback must not imply an operation succeeded before Core confirms it.
- Dragging a multi-selection respects ownership, dependencies, and confirmation/impact requirements.
- Deleting a Collection moves it and all currently contained resources, with genuinely owned descendants, to Trash as one recoverable cascading operation. Do not import legacy Archive commands: Workbench uses Trash, not a separate Archive lifecycle.

### 7.5 Navigation history and performance

Back/Forward and recent destinations are per window. Explorer uses virtualized rows for large trees, preserving UUID-based focus, selection, expansion, and active-resource reveal. Lazy content or virtualization must not make keyboard navigation or scroll-to-reveal unreliable.

## 8. Search — separate Left Panel activity

### 8.1 Purpose and boundaries

Search provides persistent discovery of resources and content in the active Workbench. It is **not** an Explorer filter. Command Center/Quick Open is a distinct fast-navigation surface with approved modes: plain text Quick Open, `>` commands, `@` Records, `#` resource types.

Search can surface eligible resources, Fields, Records, matching values, and Files, with File content search limited by actual supported File capabilities. A Search result always retains enough context to identify its resource and path; names are labels, not identity.

### 8.2 Layout

```text
SEARCH                                            …
[ Search Workbench…                           ]
18 results                             [Filters]
[Grouped | List]       [Additional options…]
──────────────────────────────────────────────────
Grouped mode:                     List mode:
▾ RESOURCES (4)                   Revenue · Table / Sales
  Revenue                         Revenue: $125k · Record / …
  Revenue Forecast                Revenue Forecast · Query / …
▾ RECORDS (12)
  Revenue: $125k
▸ FILES (2)
```

The Search header follows the common compact panel header. The query field and compact search/filter controls stay available while results scroll. Optional case/word/pattern matching depends on supported search capabilities; do not promise unsupported operators. Active filters are inspectable and removable without forcing an always-open filter panel.

### 8.3 Grouped or unified results — LP-05

Users explicitly choose either:

- **Grouped:** Collapsible categories (e.g., Resources, Records, Files, and other supported match classes) with category counts. Category expansion states are preserved.
- **Unified list:** One relevance-ranked result list across categories. Each result shows its type, match, and location/context so mixed result types remain intelligible.

Switching modes is **presentation-only**. It must not clear or rerun a query merely to change layout, drop filters, discard results, or change target identities. Search may update results for actual query/filter/data changes. Persist the user's presentation preference. Grouping is independent of search filters; no permanently visible category-filter bar is required.

**Difference from legacy:** Better Spreadsheet described Tree/List presentations with structural Explorer ordering. The new approved Workbench decision explicitly provides **grouped categories or a unified relevance-ranked list**; do not silently inherit the legacy structural ordering as the universal Search sort rule.

### 8.4 Search state and navigation

Preserve query, supported options, filters, results, grouping mode, expanded groups, selection, and useful scroll position when switching activities or hiding the panel. Search results use UUID-backed navigation:

- Resource match: navigate/open using shared Work Area commands and tab behavior.
- Field/Record/value match: navigate to the containing Table/View and reveal the appropriate data context without silently rewriting saved View definitions.
- File match: use approved File open/preview handling and report unavailable external Files appropriately.

Where a larger result presentation is needed, a transient Work Area Search Results surface may show the same active search with richer context; it does not automatically create a saved Query. If synchronized presentations are implemented, their query/filter state must remain coherent.

## 9. Inline states, errors, and recovery — LP-07

Show empty/loading/error/unavailable states **inside the affected section or results area**, keeping other sections and the header controls usable.

| State                                 | Explorer                                                                         | Search                                                                    |
| ------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| No resources / no Files               | Contextual empty text and eligible `New` / `Add File` action inside that section | Query entry remains available; initial guidance in results area           |
| No matches                            | Not an Explorer search state                                                     | `No results`, with clear/filter suggestions                               |
| Loading / updating                    | Section-local progress or skeleton without replacing unrelated sections          | Keep query/filters visible; results show loading or updating state        |
| Recoverable failure                   | Inline explanation and Retry/Repair where applicable                             | Inline search failure with Retry; preserve query and filters              |
| Missing dependency / unavailable File | Keep identifiable resource and actionable status                                 | Keep result context; explain why it cannot open and offer relevant repair |
| Partial operation                     | Local contextual outcome plus Problems/Operations entry                          | Same feedback model when navigation or search work partially fails        |

Errors use plain-language impact and next actions, color-independent indicators, and accessible announcements. Avoid duplicate toasts for persistent problems. Core remains authoritative for canonical writes, validity, and operation outcomes. Derived freshness and canonical save status are distinct.

## 10. Accessibility, responsiveness, and persistence

### 10.1 Accessibility

- Use semantic tree/treeitem/expanded/selected behavior or an equivalently accessible model; expose meaningful names, types, levels, and active state.
- Support keyboard-only navigation, selection, range/multi-selection, disclosure, context menu, creation, search, filters, grouping toggle, and panel switching.
- Distinguish keyboard focus, Explorer selection, and Work Area active resource.
- Hover actions also appear on focus and remain reachable without a pointer.
- Respect reduced motion, screen-reader announcements, and WCAG 2.2 AA targets where applicable.
- Restore focus meaningfully after dialogs, menus, resource moves, and panel toggles.

### 10.2 Constrained widths

The shell owns preferred/effective panel widths and responsive collapse priority. The Left Panel retains compact headers and scrollable content; long labels truncate with full-name access. Action controls may prioritize overflow. No horizontal overlay should unexpectedly cover the Work Area. Explicitly closed panels stay closed until intentional reveal.

### 10.3 State ownership

| State                                                                                                           | Owner / persistence intent                                                                                        |
| --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Canonical resources, hierarchy, UUIDs, mutations, Trash, File references                                        | Workbench Core; transactional persistence                                                                         |
| Active activity, panel visibility, preferred width, tree expansion, scroll, selection, active-resource UI state | UI/window/session state; restore as appropriate                                                                   |
| Explorer sort/manual presentation and Files grouping/sort                                                       | UI preferences; Files grouping/sort survive restart and reopen                                                    |
| Search query, options, filters, result grouping, collapsed categories, selection/scroll                         | Left Search view-local state; preserve through activity switching and collapse; persist preference where approved |
| Work Area active resource, tabs, preview/permanent/pinned state, editor groups                                  | Work Area/session contracts; not canonical Explorer ownership                                                     |

Persisted layout preferences are distinct from temporary effective geometry under constraints. Never serialize an invalid temporary collapse as a deliberate user width preference.

## 11. Key interaction scenarios

1. **Create within Collection:** Select `Sales`; Explorer header `+` defaults eligible new resource to `Sales`; Files and Collections still default to root.
2. **Browse Table Views:** Single-click `Customers` selects and toggles its Views; click disclosure alone toggles without selection; double-click `Customers` opens its last-used valid View permanently.
3. **Multi-select:** Ctrl/Cmd/Shift-select several resources without expanding them; invoke shared bulk command; Core validates eligibility and impact.
4. **Navigate from Search:** Open a matching View; Work Area activates it; visible Explorer automatically expands only necessary ancestors and reveals the View; if Search remains the current activity, Explorer is not forcibly shown.
5. **Change Search presentation:** Switch grouped to unified list; retain query, filters, match set, and selection where meaningful; reorder unified results by relevance.
6. **Hide and restore Left Panel:** Collapse via active Activity Bar icon; Work Area changes resources; panel remains closed; on reopening Explorer, active-resource reveal can be applied without clearing prior unrelated expansion.
7. **Files grouping:** Choose `Type` in Files overflow; categories are virtual, not folders; File ownership and attachment references do not change.
8. **Section error:** Files metadata unavailable; show an inline Files-section retry/repair state while the Workbench tree remains usable.

## 12. Cross-document responsibility and handoff

- **`ShellDesign.md`:** Physical region placement, common panel header, resizing, layout presets, responsive collapse, title/menu, Status Bar, window/session behavior.
- **`WorkAreaDesign.md`:** Tabs, preview/permanent/pinned behavior, editor groups, opening a Table/View, Record/Field reveal in editors, expanded Search Results surface.
- **`RightPanelDesign.md`:** Field/schema inspection, Record Details, contextual Notes, and feature-specific contextual editors.
- **`BottomPanelDesign.md`:** Problems, Operations, History, progress and diagnostics surfaces.
- **Product/Core specs:** Canonical data model, authorization/validation, UUIDs, resource lifecycle, file ownership, search/indexing constraints, and operation outcomes.

## 13. Deferred implementation detail (not new product decisions)

Later screen mockups and component specifications may set exact row heights, indentation steps, icon sizes, hit-target sizes, tooltip timing, animation, palette, responsive breakpoints, and virtualized-tree implementation. Search implementation may determine available operators, indexing, result limits, and File-content support within the approved product scope. These are not grounds to reopen the eight approved Left Panel design decisions.

## 14. Source cross-check

Read-only references checked against the current `ui-design` branch of [`kyliejurgs/Workbench`](https://github.com/kyliejurgs/Workbench):

- `docs/WorkbenchV1Specs.md` — Desktop & Workspace §§1A–1G; Resource Model §§2A–2G.
- `docs/SystemDesign.md` — Core/UI authority, local-first architecture, session/layout responsibilities.
- `docs/UIUXDecisionSet.md` — Systems 01, 02, 05, 07, 08, 11, 12.

Legacy references checked from [`kyliejurgs/better-spreadsheet`](https://github.com/kyliejurgs/better-spreadsheet):

- `docs/application-design/left-panel-views.md` — shared panel structure, Explorer tree/Files separation, Search patterns.
- `docs/application-design/application-layout.md` — panel relationship and Activity Bar interactions.

Legacy **Archive** commands, single-click preview-opening, automatic alphabetical-only Explorer sorting, and structural-order-only Search results are **not** adopted where they conflict with approved Workbench decisions.

**Repository modification:** None. This is a standalone Markdown design deliverable.
