# Workbench V1 — Right Panel Detailed Design

**Status:** Approved detailed design — 12 decisions locked (2026-10-19); review clarifications incorporated without changing decisions  
**Target repository path:** `docs/DetailedDesign/RightPanelDesign.md`  
**Scope:** Contextual inspector tabs, Record/Field inspection, Notes, specialized resource inspection, pinning, editing, drafts, and advanced-editor handoff.  
**Companions:** `../WorkbenchV1Specs.md`, `../SystemDesign.md`, `../UIUXDecisionSet.md`, `ShellDesign.md`, `LeftPanelDesign.md`, `WorkAreaDesign.md`, `BottomPanelDesign.md`.

## 1. Purpose and authority

The Right Panel is Workbench's contextual inspection and everyday-editing surface. It supports quick Record and Field editing, contextual Notes, resource information, and specialized inspection/configuration without displacing the active Work Area editor. Complex editing moves into an appropriate Work Area editor; focused operations may use dialogs.

- The approved product, architecture, and UI/UX baselines remain authoritative. This document refines their presentation and interaction contracts, not canonical semantics.
- Explicitly approved detailed-design decisions govern the Right Panel. The review clarifications in this document explain existing requirements; they are **not** additional or replacement decisions.
- Workbench Core owns canonical resources, typed values, validation, Notes, persistence, History, dependency analysis, and version-safe writes. UI state includes tab choice, selection following, section expansion, inspector pin presentation, focus, and transient drafts. Core-backed recovery mechanisms apply where required.
- Stable UUIDs, not display names or row positions, identify inspected targets, pinned targets, Note owners, and draft destinations.
- Legacy [`better-spreadsheet`](https://github.com/kyliejurgs/better-spreadsheet) is a reference for prior patterns only; it does not override approved Workbench behavior.
- Exact dimensions, colors, iconography, motion, and numerical breakpoints are deferred to progressive visual specifications.

## 2. Approved decision register

| ID    | Topic                | Locked decision                                                                                                                                                                                              |
| ----- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| RP-01 | Tab placement        | A — Context tabs across the top of the Right Panel; independent of Work Area tabs.                                                                                                                           |
| RP-02 | Context following    | B — Context-aware auto-follow; an explicit manual tab choice pauses automatic tab switching within the active editor resource; changing active resource resumes following.                                   |
| RP-03 | Section organization | B — Collapsible sections only; multiple sections may remain open, with sensible defaults and remembered expansion state. No secondary section navigation, section shortcuts, or section dropdown navigation. |
| RP-04 | Resource-aware tabs  | D — Tabs depend on resource type; Table View context includes Info, Record, Field, Notes; Query/Dashboard/File/Automation contexts include Info and Notes, with specialized inspectors as required.          |
| RP-05 | Inspectee pin        | B — Optional pin holds an individual Record or Field inspection target independent of Work Area tab pinning.                                                                                                 |
| RP-06 | Editing controls     | C — Hybrid: simple values click to edit; specialized controls (choices, checkboxes, dates, relationships, Notes, etc.) are directly interactive where appropriate.                                           |
| RP-07 | Cross-resource pins  | B — Pinned Record/Field inspectors remain accessible across active-resource changes without replacing normal contextual tabs.                                                                                |
| RP-08 | Pin presentation     | C — Compact pinned-inspector menu in the tab bar; clear pinned-content active indication, target identity, type, origin, and unpin controls.                                                                 |
| RP-09 | Notes target         | A — Notes automatically follow selected Record/Field, falling back to active View/resource; no Note-target selector.                                                                                         |
| RP-10 | Field layout         | A — Always stack labels above full-width values/editors; self-labeled checkboxes/toggles may remain inline. No responsive two-column field rows.                                                             |
| RP-11 | Drafts/navigation    | C — Navigation is uninterrupted; Workbench-scoped draft recovery queue distinguishes invalid, pending, and failed-save states.                                                                               |
| RP-12 | Advanced editing     | C — Substantial Record/Field editors open in Work Area tabs; focused operations use modal dialogs as appropriate; reuse existing editor tabs.                                                                |

All twelve decisions are final for this design pass. No additional decision is implied by the explanatory contracts below.

## 3. Placement, geometry, and shell integration

- The Right Panel is docked to the right of the Work Area, below the shell's header rows and above the Status Bar. It occupies the side-panel height defined by `ShellDesign.md`; the Bottom Panel is **under the Work Area only**.
- The Right Panel is **initially closed**. Ordinary selection changes do not open it. Explicit inspection/reveal commands may open it according to the shell's intent-aware reveal policy.
- Its visibility, preferred width, resizing, persistence, and responsive constraints belong to Shell Design. When horizontal space is constrained, the Right Panel yields before the Left Panel under the approved shell rules.
- The compact shared panel header provides identity, contextual actions/overflow, and a close control. Closing hides the panel; it must not discard recoverable drafts or silently unpin targets.
- The Right Panel is one shared surface for the active editor context, not a separate independently docked panel per Work Area group. It follows the active resource in the focused editor group, subject to RP-02 and pins.
- When hidden, selection and resource changes must not automatically reopen it. Reopening restores safe, meaningful inspector state where the targets remain valid.

### 3.1 Structural sketch

```text
┌───────────────────────────────────┐
│ INSPECTOR                 …    ×   │  shared compact panel header
├───────────────────────────────────┤
│ Info  Record  Field  Notes   📌 ▾  │  resource-aware context tabs + pin menu
├───────────────────────────────────┤
│ Target title / identity / status  │
│ ▾ Overview                        │
│   Field label                     │
│   [full-width value or editor]    │
│ ▾ All Fields                      │
│   ...                             │
│ ▸ Relationships                   │
│ ▸ Attachments                     │
│                                   │
│ [context actions / open editor]   │
└───────────────────────────────────┘
```

The sketch is conceptual, not a prescribed width or a mandate to duplicate the same content across tabs and sections.

## 4. Resource-aware inspector tabs — RP-01, RP-02, RP-04

### 4.1 Context tabs

- **Table View editor:** `Info`, `Record`, `Field`, `Notes`. `Record` and `Field` show the current applicable selection or a helpful select-an-item state. `Info` is a genuine resource overview, never a fallback embedded inside Record or Field.
- **Query, Dashboard, File, Automation editors:** `Info`, `Notes`, plus contextual specialized inspectors where required. Do not display meaningless Record or Field tabs simply to keep a uniform strip.
- **Other supported resources/tools:** Show only relevant tabs and any specialized inspectors required by the approved feature set. Resource information and contextual Notes remain discoverable where applicable.
- Tab order and labels should be stable within each resource context. The context tabs are **not** Work Area tabs; selecting one does not open or close a Work Area editor.
- The contextual `Info` tab summarizes the active resource, including relevant identity, metadata, status, dependencies, and navigation/actions when supported. Specialized tabs or sections may provide deeper configuration.

### 4.2 Auto-follow and manual choice

- With no manual override, selecting a Record or Field ordinarily activates the corresponding inspector tab when available.
- Manually choosing a contextual inspector tab pauses **automatic tab switching** while the same editor resource remains active. Subsequent selections in that resource do not forcibly switch tabs.
- Changing the active editor resource clears that manual tab override and resumes resource-appropriate following.
- The inspector's displayed target within a selected tab may still update with selection; manual **tab** choice is not a pin and does not freeze the inspected target.
- If a tab becomes inapplicable on a resource change, activate a valid contextual tab, normally `Info`, without showing a stale or misleading inspector.
- Selection changes never reopen a hidden Right Panel.

### 4.3 Empty and inapplicable contexts

- `Record` and `Field` tabs remain discoverable in Table View context even with no suitable selection; provide clear prompts rather than inventing a selected item.
- Loading, empty, unavailable, stale, permission/trust-limited, invalid, and error states appear locally in the affected inspector content with appropriate recovery actions.
- Avoid turning transient selection changes into global alerts; reserve prominent messaging for actionable failures or risks.

## 5. Collapsible section model — RP-03

- Organize each inspector into a vertical sequence of **collapsible sections**; allow more than one section expanded at once.
- Provide sensible initial expansion defaults and remember section expansion per meaningful inspector context, restoring safely when a target is revisited.
- Use clear section headings and contextual actions/overflow. Keyboard and screen-reader users can expand, collapse, and operate section controls.
- **Do not introduce** a secondary section tab strip, shortcut palette, section dropdown navigator, or other section-level navigation system.
- When content is extensive, allow contextual searching/filtering **within** the inspector where already required (notably searchable Record Details). This is content search, not an alternate section navigator.
- Scrolling is vertical within the inspector content. Ordinary fields do not require horizontal scrolling.

## 6. Record inspection and editing

### 6.1 Record Details coverage

The Record inspector must provide the approved Record Details capabilities: **Overview, All Fields, Relationships, Attachments, Notes, and Activity**, with optional Field grouping and searchable Record content. These are functional content groupings, not a requirement for a second navigation system. Where the dedicated `Notes` tab already provides the same Record Note, a section-level entry may link or reveal that existing Note rather than create a second independent Note editor.

- **Overview:** record identity/display label, important summary values, relevant status and common actions.
- **All Fields:** searchable typed Field/value presentation, respecting applicable read-only, computed, generated, or constrained values.
- **Relationships:** linked Records, reverse navigation, picker access, relevant relationship status and policies; never imply linked target Records are owned by the current Record.
- **Attachments:** linked File associations, preview/open actions, attach/detach flows through the approved File model. Detaching a File does not automatically delete it.
- **Notes:** the single optional Note owned by the Record, following RP-09 and the approved Note lifecycle.
- **Activity:** relevant History/activity information and navigation to fuller History where needed; do not invent a second canonical audit store.

### 6.2 Editing conventions — RP-06, RP-10

- **Always-stacked fields:** a visible label sits above the value or full-width editing control, regardless of panel width. There is no responsive two-column form layout.
- A self-labeled checkbox or toggle may remain inline, without a redundant stacked label.
- Simple text-like values present a compact readable value until clicked/focused for editing. Specialized controls may remain directly interactive: choice selectors, date/time pickers, relationship pickers, checkboxes, attachment controls, and Note editing.
- Long values wrap or expand appropriately; editing controls use the available content width. Never truncate an edit value in a way that hides its meaning without a way to inspect it.
- Render Field types with their approved semantics: formatted display vs canonical typed value, blank vs zero/false, configured currency, percentage storage, time/DateTime distinctions, and explicit read-only calculated/generated values.
- Show validation at the affected control with actionable messages. Errors block new commits; warnings may permit them and remain discoverable in Problems.
- Advanced multi-step or structural operations use guided flows and previews instead of forcing complex interactions into narrow field controls.

## 7. Field inspection and schema safeguards

- The Field inspector provides everyday Field information and operations: label/identity, type, display settings, supported defaults, constraints, and relevant dependencies, with controls appropriate to the Field's capabilities.
- Distinguish canonical Table Field order from View-specific Field order and visibility. Do not silently mutate one when editing the other.
- Advanced structural work belongs in the **Schema Manager** or appropriate Work Area editor. Field type changes require preview of conversion results, incompatible/lossy values, dependencies, and impacts before applying the approved Core operation.
- Preserve recoverable originals where required. Do not silently coerce incompatible values, erase pre-existing links, or overwrite canonical data from an invalid draft.
- Calculated/generated Field values are normally read-only; supported conversion to values must be explicit.
- Field identity and pin/draft targeting use UUIDs even after rename, reorder, or movement.

## 8. Pinned inspectors — RP-05, RP-07, RP-08

### 8.1 Pinning contract

- A Record or Field inspector can be pinned to a specific **UUID-identified target**. The pin freezes the inspected target, not the active Work Area selection or editor resource.
- Pins are independent of Work Area tab pinning and do not make the target a new Explorer resource.
- Pinned inspectors remain available across changes to the active Work Area resource. They **do not replace** the new resource's normal `Info`, `Notes`, or other contextual tabs.
- Ordinary selection of another Record or Field updates the corresponding unpinned contextual inspector. It does **not** change or release an existing pinned target.
- A user can switch between normal contextual tabs and a pinned inspector without altering Work Area selection. The pinned inspector shows the target's origin/resource context clearly.

### 8.2 Pinned-inspector menu

- Use a **compact pin-menu control** in the context tab bar, not an always-visible second row or an expanding strip of pinned tabs.
- The menu lists each pinned target's disambiguated display name, target type, originating resource, and an unpin action. Names are for display; UUIDs determine identity.
- Selecting an entry displays that pinned inspector in the Right Panel content area; the pin-menu control visibly indicates that pinned content is active.
- Contextual tabs remain accessible and selecting one returns to ordinary resource inspection.
- Handle renamed, deleted/trashed, inaccessible, or otherwise unavailable targets with explicit status and appropriate navigation/recovery, rather than silently redirecting to another object with the same name.
- Hiding the Right Panel does not implicitly release pins. Persistence beyond safe window/session restoration follows approved shell state semantics; no independent canonical pin ownership is introduced.

## 9. Contextual Notes — RP-09

### 9.1 Automatic Note ownership selection

- The `Notes` tab **automatically follows** the selected Record or Field when applicable.
- If no applicable Record/Field selection exists, it falls back to the active View or active resource, as appropriate to that editor's context.
- There is **no Note-target selector**, manual Note-target override, or independent Note pinning. This is separate from RP-02: manually choosing the **Notes tab** can keep that tab active while its Note owner continues to follow selection.
- Context is resolved by parent **type and UUID**, not by display name. A Note is optional and there is at most one Note per supported parent.
- Supported Note owners remain Workbench, Collection, Table, Field, Record, View, Query, Dashboard, Automation, and File. This decision does not remove the ability to reach those Notes through their applicable resource contexts and navigation.
- A missing Note presents an empty/create state associated with the correct owner; it does not create an unrelated standalone Note resource.

### 9.2 Note editor capabilities

- The Right Panel is the **primary** Note editor. Support Markdown Source and Live Preview modes, headings, checklists, an outline, standard links, and UUID-based internal resource links with appropriate resolution/repair behavior.
- Provide the approved editing shortcuts, autosave, recoverable drafts, History, and Undo/Redo. A full Note editor tab may be opened for more space without creating a second Note identity.
- When selection changes, retain and safely recover incomplete/invalid/pending Note drafts for their original parent. Never silently reassign the text to the new Note owner.
- Notes are parent-owned supporting objects, not independent Explorer resources; no separate Note move, duplication, ordering, pinning, or reassignment behavior is introduced. No Cell Notes.
- Parent rename/move preserves Note identity; deletion/restoration follows the parent's Core-managed lifecycle.

## 10. Specialized inspector content

The resource-aware tab model must preserve already-approved specialized capabilities, without inventing fixed tab sets before their resource-specific designs are completed.

- **Views:** advanced View configuration where appropriate, distinguishing saved configuration from temporary overrides; relevant grouping, summaries, formatting, density, and view-specific options. Frequent commands may stay in the Work Area toolbar/popovers.
- **Queries:** dependency information and impact navigation, freshness/stale/error context, parameters or diagnostics where appropriate. Query definition/results editing remains in the Work Area; inspector content does not create a second Query authoring mode.
- **Dashboards:** contextual widget inspection/configuration, source mapping, filters, freshness, dependencies, and repair where applicable. Respect explicit View/Edit modes and staged Dashboard layout changes with Save and recoverable drafts.
- **Files:** identity, embedded/external status, usage/attachment relationships, relink and relevant actions, while keeping primary File editing/preview in the Work Area.
- **Automations:** contextual trigger/condition/action properties, configuration and execution status, validation/trust review, and dependencies. Respect imported/untrusted activation safeguards; run logs/Operations are not replaced by the inspector.
- **Settings and other system surfaces:** contextual settings may appear when required by the approved Settings architecture, synchronized with their authoritative setting scope.

Specialized content uses the same collapsible sections, stacked-field layout, keyboard/accessibility standards, and draft/validation rules. `Info` and `Notes` retain their resource-aware roles.

## 11. Drafts, validation, conflicts, and navigation — RP-11

### 11.1 Uninterrupted navigation

- Changing Record, Field, resource, or inspector tab is not blocked solely because an edit is invalid, incomplete, or awaiting a save.
- Valid edits autosave through approved Core operations. Invalid edits remain **drafts**, never canonical data. Warnings and errors retain their approved severity semantics.
- Drafts are associated with stable target identities, relevant Field/Note identity, and sufficient editing context for accurate restoration. A selection change must not transfer a draft to the newly selected target.
- Preserve drafts during panel hide/show, applicable tab switches, and interrupted sessions according to approved recovery rules.

### 11.2 Workbench-scoped recovery queue

- Provide a compact Right Panel access point for a **Workbench-scoped draft recovery queue**. It lists affected resource/target and Field or Note context, with direct return/navigation where possible.
- Distinguish **invalid drafts**, **pending saves**, and **failed saves**. Also surface **version/concurrency conflicts** with their own actionable resolution state; do not mislabel every condition as invalid input.
- For conflicts, preserve the local draft, show the current saved value where appropriate, and require explicit resolution. Never silently overwrite a newer canonical version.
- A pending save should not be represented as a committed success. Failed saves remain actionable and recoverable. Revalidate recovered drafts before commit.
- Draft status and controls must be keyboard/screen-reader accessible; status is not conveyed through color alone.
- The queue complements the Bottom Panel's **Problems**, **Operations**, and **History**; it does not replace their broader reporting, operation tracking, or canonical History/Undo responsibilities.

## 12. Advanced editor and modal handoff — RP-12

- Quick inspection and everyday edits stay in the Right Panel.
- **Substantial Record/Field editing** opens an appropriate full Work Area editor tab, including advanced Record editing or Schema Manager as required. Preserve the originating inspector's context and meaningful state.
- Reuse an already-open matching editor tab in the appropriate group when possible, following `WorkAreaDesign.md` duplicate-tab, preview/permanent/pinned, focus, and navigation rules. Do not introduce an independent editor-tab system in the Right Panel.
- **Focused operations** may use modal dialogs: confirmations, linked-Record pickers, impact/conversion reviews, and other bounded tasks with a clear outcome. Do not use a modal as the default container for extended multi-section editing.
- A dialog must expose Cancel/confirm and validation/impact feedback as appropriate; preserve safe drafts on interruption. Destructive operations use approved Core lifecycle and dependency safeguards.
- Full editor, modal, and inspector refer to the **same canonical identities** and Core-backed data. Concurrent edits use version-safe updates and explicit conflict handling. Do not imply that closing an editor commits an invalid draft.
- The optional full Note editor follows the approved Note behavior and the same Work Area navigation conventions.

## 13. Interaction, state, and accessibility contracts

### 13.1 State and restoration

- UI-managed state includes active contextual tab, manual-tab override scoped to the current editor resource, section expansion, pin-menu presentation, active pinned inspector, scroll/focus, and panel visibility/geometry under Shell Design.
- Canonical resources, Notes, Field values, settings with approved persistence scope, and History remain Core-owned. Recoverable draft persistence follows the approved recovery architecture rather than being reduced to transient component state.
- Restore state by UUID where possible; gracefully fall back when targets no longer exist or cannot be inspected. Never silently attach stale state to a different resource with the same label.
- Changes to active Work Area group/resource update ordinary inspector context. Pins remain independently accessible; manual tab override resets on active-resource change.

### 13.2 Input and accessibility

- All tabs, section disclosures, menus, pins, field editors, draft-recovery entries, and advanced-editor actions support keyboard-only operation and visible focus.
- Announce contextual target/type, selected tab, expanded/collapsed state, validation feedback, save state, and pinned context to assistive technology.
- Use readable labels and semantic control relationships; do not rely solely on icons or color for state.
- Modal dialogs use accessible focus management and restoration. Avoid surprise focus jumps on ordinary selection changes.
- Target WCAG 2.2 AA where applicable, consistent with the global UI/UX baseline.

## 14. Cross-document alignment and explicit non-goals

| Concern                                                         | Owning design / authority                                                | Right Panel contract                                                                       |
| --------------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| Panel visibility, preferred width, responsive collapse, presets | `ShellDesign.md`                                                         | Uses shared shell rules; does not define competing geometry.                               |
| Explorer selection, Search, explicit reveal                     | `LeftPanelDesign.md`                                                     | Responds to active context without forcing Left Panel navigation or opening hidden panels. |
| Work Area tabs, groups, advanced editors, toolbars              | `WorkAreaDesign.md`                                                      | Opens/reuses appropriate Work Area editors; pinned inspectors are not Work Area tab pins.  |
| Problems, Operations, History                                   | `BottomPanelDesign.md` / approved UI/UX baseline                         | Provides contextual status and recovery links without duplicating these system surfaces.   |
| Typed values, Notes, validation, identity, concurrency, History | `WorkbenchV1Specs.md` and `SystemDesign.md`                              | Uses Core authority; does not redefine canonical semantics.                                |
| Legacy UI patterns                                              | [`better-spreadsheet`](https://github.com/kyliejurgs/better-spreadsheet) | Informational only; approved Workbench behavior prevails.                                  |

**Explicit non-goals:** a secondary inspector section navigator; an independent Note target selector; standalone Note pinning; always-visible pinned-inspector tab rows; responsive two-column field layouts; modal-only advanced editors; navigation-blocking invalid-draft prompts; a second canonical data/draft/History system; automatic opening of a hidden Right Panel on ordinary selection.

## 15. Review clarifications incorporated (no new decisions)

The post-decision requirements review found **no direct contradictions** among RP-01 through RP-12. These existing requirements are made explicit here:

1. **Record Details coverage:** searchable Overview, All Fields, Relationships, Attachments, Notes, and Activity capabilities coexist with collapsible-only section navigation (Sections 5–6).
2. **Specialized inspectors:** View, Query, Dashboard widget, File, Automation, and contextual Settings inspection coexist with resource-aware tabs (Sections 4 and 10).
3. **Notes functionality:** automatic Note ownership selection does not remove the approved Markdown editor, outline, link, History, and draft features (Section 9).
4. **Draft/conflict behavior:** recovery queue status differentiates invalid, pending, failed, and conflict cases; version-safe resolution remains mandatory (Section 11).
5. **Pinned vs. ordinary inspection:** a pinned target remains fixed while the ordinary contextual inspector continues following selection; both remain accessible (Section 8).

These clarifications preserve the twelve locked choices and the existing product/architecture requirements. Resource-specific screen layouts, exact visual tokens, and Bottom Panel presentation remain subjects for their own detailed designs.
