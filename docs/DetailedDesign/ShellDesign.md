# Workbench V1 — Shell Detailed Design

**Status:** Detailed shell decisions 1–9 approved (2026-10-18)  
**Scope:** Shared application chrome, spatial arrangement, panel relationships, global commands, responsive behavior, and window/session coordination.  
**Baseline:** `docs/WorkbenchV1Specs.md`, `docs/SystemDesign.md`, and `docs/UIUXDecisionSet.md` on the `ui-design` branch.  
**Companion detailed designs:** `LeftPanelDesign.md`, `WorkAreaDesign.md`, `RightPanelDesign.md`, `BottomPanelDesign.md`.

## 1. Purpose and authority

Workbench is a compact, spreadsheet-forward, local-first desktop Data IDE. This document refines the already-approved application-shell requirements; it does not replace the product or architecture baselines. The nine explicitly approved detailed shell decisions are recorded below. Other behavior in this document carries forward previously approved requirements. Where exact visual tokens, numerical dimensions, or surface-specific behavior are not yet approved, this document does not invent them.

The React/TypeScript shell coordinates window-local navigation, commands, focus, editor groups, panel visibility and geometry, and session presentation state. Workbench Core remains authoritative for canonical domain data, validation, persistence, operations, and History. UI layout state is not canonical resource data.

## 2. Approved shell composition

The application shell contains:

- A **title bar** with Workbench/window identity, integrated Command Center, and native window controls.
- A **second-row application menu bar** with File, Edit, View, and other applicable menus.
- An **optional, separately configurable Command Bar**, hidden in the default layout. This is distinct from the integrated Command Center in the title bar.
- A **reorderable Activity Bar** for Left Panel activities, with eligible entries configurable and essential navigation remaining discoverable.
- A **Left Panel** for one activity at a time, initially Explorer; Search is a separate Left Panel activity, not an Explorer filter.
- A **tabbed Work Area** with one editor group by default and a maximum of two groups, split horizontally or vertically.
- A **contextual Right Panel**, initially closed.
- A **Bottom Panel** for Problems, Operations, and History, initially closed.
- A **persistent Status Bar** spanning the bottom of the window.

### 2.1 Physical arrangement — Decision 1 (locked: A)

The Bottom Panel occupies the area **directly beneath the Work Area only**. The Left and Right Panels retain their full available height between the header region and the Status Bar. Side panels dock and push the Work Area; they do not unexpectedly overlay the editor.

```text
+-----------------------------------------------------------------------+
| Title bar: Workbench identity | Command Center | Window controls      |
+-----------------------------------------------------------------------+
| Application menus: File | Edit | View | ...                           |
+-----------------------------------------------------------------------+
| Optional Command Bar (hidden by default)                              |
+-----+----------------+--------------------------------+---------------+
|     |                |                                |               |
| Act | Left Panel     | Work Area                      | Right Panel   |
| ivi | (Explorer or   | tabs / one or two editor      | (contextual)   |
| ty  | another        | groups                         |               |
| Bar | activity)      +--------------------------------+               |
|     |                | Bottom Panel (Work Area only)  |               |
+-----+----------------+--------------------------------+---------------+
| Persistent Status Bar                                                 |
+-----------------------------------------------------------------------+
```

The diagram illustrates the structural relationship, not pixel proportions. Hidden panels yield their space to the remaining layout.

### 2.2 Header arrangement — Decision 2 (locked: custom two-row)

The **first row/title bar** contains the Workbench identity, integrated Command Center, and platform window controls. The **second row** contains the application menus (File, Edit, View, and other relevant menus). The optional Command Bar is a distinct command surface, not a substitute for either row and not displayed by default. Commands exposed through menus, Command Center, Command Bar, shortcuts, context menus, and toolbars use the shared command registry and consistent context-sensitive enablement.

### 2.3 Global shell appearance — Decision 8 (locked: B + C hybrid)

Use a **compact layered professional IDE** aesthetic: distinct title/menu, navigation, editing, and status surfaces with restrained contrast and subtle separators, combined with dense, efficient spacing and minimal decorative chrome. The Work Area has visual and spatial priority. Compactness must not compromise legibility, focus visibility, keyboard access, or usable targets. Light, Dark, and System appearance support remains as approved. Exact palette, typography, spacing tokens, icon sizes, and dimensions remain open for progressive visual review; examples are not normative measurements.

## 3. Panel geometry and interaction

### 3.1 Docking and preferred dimensions — Decision 3 (locked: C)

Left, Right, and Bottom Panels support visibility toggles and resizing. Remember user-chosen **preferred** dimensions separately from **effective** dimensions imposed by the current window constraints. As space decreases, temporarily constrain or collapse panels to preserve the minimum viable Work Area. As space returns, restore the preferred dimensions and requested visibility. Never persist a temporary constraint as the user's new preference. The Work Area has layout priority. Numeric minimums, maximums, resize-handle sizes, and breakpoints are to be set during detailed visual/implementation specification, not presumed here.

### 3.2 Left Panel activity switching — Decision 4 (locked: A)

Only **one Left Panel activity is displayed at a time**. Choosing Search replaces Explorer in the existing Left Panel, rather than stacking or opening another navigation column. Returning to Explorer restores its previous selection, tree expansion, and scroll position where still valid. Each activity can retain appropriate local presentation state. Selecting the currently active Activity Bar entry can toggle the corresponding Left Panel visibility, consistent with the approved Activity Bar behavior. The Activity Bar supports reordering, show/hide of eligible entries, and reset; essential entry points must remain discoverable.

### 3.3 Intent-aware panel reveal — Decision 5 (locked: C)

An **explicit destination command**, such as Inspect Field, Show Problems, or Show Operations, reveals the corresponding panel and leaves it open. Ordinary selection, editor focus, or context changes update panel content but **do not reopen** a panel the user deliberately closed. A command that intentionally opens a panel should also direct the user to the requested surface with meaningful keyboard focus. Closing or hiding a panel must not discard unrelated drafts, tabs, or canonical data. Explicit reveal and responsive constraints are reconciled under §3.5.

### 3.4 Shared panel headers — Decision 9 (locked: A)

Left, Right, and Bottom Panels use a consistent **compact header framework**: predictable title position, contextual action area, overflow actions, and close/collapse control. Individual panels provide their own commands and navigation within that framework; the Bottom Panel may expose Problems/Operations/History navigation in its header. Panel headers and dividers offer discoverable, keyboard-accessible controls and resize affordances. Exact icons and header dimensions remain visual-design details.

### 3.5 Responsive collapse order — Decision 7 (locked: A)

When horizontal space becomes insufficient:

1. Constrain the side panels toward their allowed minimum widths.
2. Temporarily collapse the **Right Panel first**.
3. Collapse the **Left Panel** only if more space is required.
4. Restore requested visibility and preferred widths when space becomes available.

An explicitly revealed panel receives temporary priority to remain usable for the invoked task; this must not silently rewrite saved visibility or sizing preferences. When the constraints make simultaneous display impossible, preserve editor usability and provide a discoverable way to reach the requested panel. Bottom Panel vertical constraints similarly preserve a viable Work Area; no additional unapproved automatic-collapse ordering is prescribed for vertical pressure. Constrained geometry is transient and must not oscillate unpredictably as space changes.

## 4. Layout presets — Decision 6 (locked: A)

Presets are **starting arrangements**, not restrictive modes. Users may adjust panel visibility and sizes after applying one. Applying a preset does not close tabs, discard drafts or editor-local state, or automatically create a second editor group; an existing two-group arrangement is preserved subject to available space.

| Preset       | Left Panel       | Right Panel | Bottom Panel | Editor groups                            |
| ------------ | ---------------- | ----------- | ------------ | ---------------------------------------- |
| **Default**  | Explorer visible | Hidden      | Hidden       | Preserve existing; one on initial launch |
| **Focus**    | Hidden           | Hidden      | Hidden       | Preserve existing                        |
| **Inspect**  | Hidden           | Visible     | Hidden       | Preserve existing                        |
| **Analysis** | Explorer visible | Visible     | Visible      | Preserve existing                        |

The title/menu rows and persistent Status Bar remain available in these presets. The optional Command Bar remains governed by its independent visibility preference unless an explicitly approved later design specifies otherwise. Presets do not destroy saved user preferences for panel dimensions. Focus layouts must remain recoverable without losing navigation or data state.

## 5. Work Area integration

The Work Area is the primary content surface and receives remaining space after applicable chrome and docked panels. It supports preview, permanent, and pinned tabs; at most two independently focused editor groups; horizontal or vertical splitting; and safe restoration of local editor state. Focused editor/group determines contextual command enablement and Right Panel content. Tabs and editors can be manipulated without mutating canonical Core state except through validated domain commands. Detailed tab behavior, drag/move, group controls, editor-specific toolbars, and content layouts belong in `WorkAreaDesign.md`.

## 6. Navigation and command integration

Explorer provides hierarchical resource browsing; dedicated Left Search provides persistent resource/content discovery; Command Center/Quick Open provides rapid navigation and command entry. Approved Command Center modes include plain text Quick Open, `>` commands, `@` Records, and `#` resource types. Explorer does not gain an independent search/filter input. The Explorer `+` creates eligible resources relative to the selection: within the selected Collection or the Collection containing the selected resource, otherwise at Workbench root, subject to root-only Files, Table-owned Views, and non-nestable Collections. This creation rule is carried forward from the approved UI/UX Decision Set and elaborated in `LeftPanelDesign.md`.

The command registry unifies menu actions, context menus, keyboard shortcuts, toolbars, and panel actions. Commands use the active window, focused group, selected resource, and other appropriate context. Shortcuts are configurable with conflict handling. Back/Forward and recent navigation remain window-local.

## 7. Status, feedback, and long-running operations

The Status Bar is persistently accessible and exposes relevant Core state, operation status, Problems, active resource, freshness, and contextual counts. Critical indicators must not be removable through customization. Distinguish canonical save state from derived Fresh/Stale/Computing/Error state; a committed write is not the same as a completed derived computation.

Use quiet feedback for routine edits, contextual actionable validation for errors, restrained toasts for meaningful events, Problems for persistent issues, and Operations for longer-running work. History remains available through its approved Bottom Panel surface. Notifications and diagnostics should link to the relevant resource, operation, or issue without duplicate noisy alerts. Error and recovery states must preserve valid editor-local state and recoverable drafts when safe. Detailed panel layouts and workflows belong in `BottomPanelDesign.md` and the relevant feature design.

## 8. Windows, persistence, and restoration

Each application window displays either Home or one Workbench. Multiple windows may share a Workbench Core while maintaining independent UI focus, navigation, layout, and editor/session presentation state. Opening an already-open Workbench must reuse/activate its existing writable authority rather than create a competing writer; new Workbenches normally open in a new window.

Restore previous windows, tabs, editor groups, active activities, panel visibility, panel dimensions, and meaningful editor-local state by default, subject to safe recovery. Distinguish application/global defaults, per-Workbench layout preferences, per-window session state, and temporary effective geometry. Persisted session/layout state must not be mistaken for canonical Workbench resource data. Handle missing resources, interrupted operations, and incompatible/restored layout states safely rather than silently discarding canonical work.

## 9. Accessibility and cross-surface behavior

The shell must support keyboard-only navigation between title/menu commands, Activity Bar, panels, editor groups, and Status Bar; visible focus and meaningful focus restoration; screen-reader context for panels and commands; accessible menus, dialogs, resizing controls, and notifications; reduced-motion preferences; and WCAG 2.2 AA targets where applicable. Color must not be the sole carrier of status. The shared shell remains usable in compact, typical, and large desktop windows, including constrained layouts and two-group configurations. No mobile-app scope is introduced.

Focus changes should be deliberate: explicit commands move focus to their target; passive context updates do not steal it. Panel hiding, switching, and responsive collapse should preserve or sensibly restore focus without losing pending drafts.

## 10. Ownership boundaries and follow-on design

| Surface / concern                                                                                  | Detailed owner         |
| -------------------------------------------------------------------------------------------------- | ---------------------- |
| Activity Bar entries, Explorer, Search, navigation trees, creation entry points                    | `LeftPanelDesign.md`   |
| Tab strips, editor groups, editor toolbars, content and editor focus behavior                      | `WorkAreaDesign.md`    |
| Contextual inspector, Field/Record editing and other Right Panel content                           | `RightPanelDesign.md`  |
| Problems, Operations, History, Bottom Panel content and navigation                                 | `BottomPanelDesign.md` |
| Shared region placement, chrome, panel coordination, responsive rules, presets, status integration | **This document**      |

## 11. Approved detailed shell decision register

| ID    | Decision                   | Locked outcome                                                                  |
| ----- | -------------------------- | ------------------------------------------------------------------------------- |
| SD-01 | Physical shell arrangement | Integrated IDE: Bottom Panel beneath Work Area only; side panels full-height    |
| SD-02 | Header arrangement         | Two rows: title bar with identity/Command Center/window controls; menus below   |
| SD-03 | Panel sizing               | Preferred dimensions with adaptive temporary constraints                        |
| SD-04 | Left Panel activities      | One visible activity at a time; restore activity state                          |
| SD-05 | Panel reveal               | Explicit commands open and retain panels; passive context changes do not reopen |
| SD-06 | Presets                    | Default, Focus, Inspect, Analysis arrangements in §4                            |
| SD-07 | Responsive collapse        | Right Panel first, then Left; explicit reveal temporarily prioritized           |
| SD-08 | Global appearance          | Hybrid layered professional IDE + high-density utility                          |
| SD-09 | Panel headers              | Consistent compact shared header framework                                      |

All nine decisions are approved. No further substantive shell-level decision is required before proceeding to `LeftPanelDesign.md`. Numerical dimensions, final tokens, and individual surface controls remain intentionally open for later detailed design and visual review.
