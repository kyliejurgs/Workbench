# Workbench — Work Area Detailed Design

**Status:** Approved detailed design — 15 decisions locked (2026-10-19)  
**Target repository path:** `docs/DetailedDesign/WorkAreaDesign.md`  
**Scope:** The tabbed central Work Area, editor groups, headers, navigation, tab interaction, resizing, and editor-level feedback.  
**Companions:** `docs/WorkbenchV1Specs.md`, `docs/SystemDesign.md`, `docs/UIUXDecisionSet.md`, `docs/DetailedDesign/ShellDesign.md`, `docs/DetailedDesign/LeftPanelDesign.md`, and forthcoming Right/Bottom Panel designs.

## 1. Purpose and authority

The Work Area is the central resource-editing surface of Workbench's compact, data-first desktop IDE. It hosts Views, Queries, Dashboards, Files, tools, and other approved editors. This document refines presentation and interaction; it does not redefine canonical data ownership, Core operations, or resource-specific editing behavior.

The V1 product and UI/UX baselines remain authoritative for functionality. Later explicit detailed-design decisions supersede earlier conflicting presentation choices; unresolved contradictions must be raised, not silently decided. The UI owns tabs, focus, drag feedback, ephemeral editor state, and layout. Workbench Core owns resource identity, saved View definitions, mutations, validation, History, and persistence. Each Workbench remains a local-first `.wkbn` database.

## 2. Decision register

| ID    | Topic                  | Locked decision                                                                                           |
| ----- | ---------------------- | --------------------------------------------------------------------------------------------------------- |
| WA-01 | Tab bar layout         | C — compact single-line tabs with dedicated context breadcrumb row                                        |
| WA-02 | Editor header          | A — dedicated resource toolbar beneath breadcrumb                                                         |
| WA-03 | Editor groups          | B — side-by-side or stacked; user-selected orientation; maximum two                                       |
| WA-04 | Tab overflow           | B — scrollable tabs plus searchable open-tabs menu per group                                              |
| WA-05 | Tab state visuals      | A — italic preview, regular permanent, pin icon for pinned                                                |
| WA-06 | Empty editor           | Hybrid B/C — compact primary actions and up to five recent resources                                      |
| WA-07 | Active group           | B — subtle header accent and slightly stronger header background; **fully approved at integrated review** |
| WA-08 | Group controls         | A — visible split control and overflow at tab bar's right end                                             |
| WA-09 | Breadcrumb interaction | D — direct navigation for openable ancestors; contextual menu otherwise                                   |
| WA-10 | Group divider          | D — VS Code-style thin draggable divider, no visible grip                                                 |
| WA-11 | Tab drag feedback      | VS Code-style — editor-area highlight, tab-bar insertion marker, edge split target                        |
| WA-12 | Close/dirty visuals    | A — VS Code-style close affordances and true uncommitted-change dot                                       |
| WA-13 | Breadcrumb overflow    | Hybrid A/B — middle truncation with clickable ellipsis menu, no dropdown arrow                            |
| WA-14 | Toolbar overflow       | Hybrid A/B — remove labels first, then overflow lower-priority actions                                    |
| WA-15 | Editor states          | B+ — contextual inline loading/errors, full-editor fallback when resource cannot open                     |

**Integrated review:** Overall Work Area layout approved. WA-07's previously provisional treatment was explicitly locked as Option B. No Work Area decisions remain provisional.

## 3. Structural layout

### 3.1 Work Area and editor groups

- The Work Area fills the central shell region and receives layout priority under constrained dimensions.
- One editor group is the default. A maximum of two groups may be open.
- Two groups may be arranged **side-by-side** or **stacked**, with an explicit user-selectable orientation. Orientation is not silently switched by responsive behavior.
- Each group independently owns its tab strip, selected editor, breadcrumb, resource toolbar, focus context, and meaningful view-local state.
- A thin resizable divider separates groups. Group widths/heights and orientation persist with window/session state, subject to minimum usable dimensions and safe restoration.
- Closing a group follows the existing safe tab/draft and session behavior. Never silently discard tabs, drafts, or operations.
- The same View may appear in both groups; within one group there must not be duplicate tabs for the same View/resource identity.

### 3.2 Editor header stack

From top to bottom, **inside each group**:

1. **Compact tab strip:** tabs, scroll/overflow navigation, visible split affordance, and group action overflow.
2. **Breadcrumb row:** active resource's hierarchical path, smart ancestor interaction, responsive truncation.
3. **Dedicated resource toolbar:** context-specific primary commands and overflow.
4. **Editor content:** resource-specific presentation, selection, editing, and localized status/recovery.

The breadcrumb is for resource context/navigation; group-management controls belong in the tab bar, not the breadcrumb. The resource toolbar remains in a predictable position but its controls differ by editor type.

Illustrative hierarchy:

```text
┌───────────────────────────────────────────────────────────────────┐
│ [Customers · Grid] [Revenue Query]    [tabs list] [split] [···]  │
├───────────────────────────────────────────────────────────────────┤
│ Workbench  ›  Sales  ›  Customers  ›  Grid                       │
├───────────────────────────────────────────────────────────────────┤
│ Filter   Sort   Group                               Settings ···  │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│                        ACTIVE EDITOR CONTENT                      │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

This diagram is structural, not a pixel-perfect component specification.

## 4. Tabs and open resources

### 4.1 Tab identity and classes

- Views, Queries, Dashboards, Files, tools, and other approved editor resources use the established **preview**, **permanent**, and **pinned** tab model.
- Opening a Table resolves to its last-used valid View, falling back to its required default View.
- Opening an already-open View activates its existing tab in the target group rather than creating a duplicate.
- Tab labels use resource icons and disambiguated names when collisions occur; hover/accessibility information can provide fuller paths.
- A **preview** tab has an *italic* label; a **permanent** tab uses a regular label; a **pinned** tab includes a pin icon. The active-tab state is independent of these classes.
- Tabs support activation, reordering, movement between groups, appropriate movement to a separate window where supported, closing, reopening/history, and safe session restoration.
- Tab pinning and preview promotion follow the already-approved tab lifecycle. Visual indicators do not alter resource persistence.

### 4.2 Overflow and navigation

- Tabs remain on a compact, horizontally **scrollable** single-line strip.
- Each group has a **searchable open-tabs menu** listing all tabs in that group, with resource icons, disambiguated names, and active state.
- Activating a tab keeps it visible in the strip where practical.
- The open-tabs menu does not replace global Quick Open, Search, or navigation history.
- Keyboard traversal, menu operation, and focus indicators are supported.

### 4.3 Closing, pinned protection, and dirty state

- Close buttons appear on the **active**, **hovered**, or **keyboard-focused** eligible tab.
- Genuine uncommitted edits are indicated by a **dot** in place of the close affordance; hovering/focusing reveals the close action.
- Do **not** mark a tab dirty simply because derived results are stale, calculations are running, or an Operation is in progress.
- Closing with uncommitted changes uses appropriate save/discard/cancel or recoverable-draft safeguards for that editor. Canonical committed resource changes remain subject to Workbench's established autosave/Core semantics.
- Pinned tabs remain explicitly closable but protected from accidental closure gestures; middle-click closes eligible non-protected tabs.
- Closing/reopening must preserve or safely resolve meaningful editor-local state, drafts, and independent ongoing Operations according to existing requirements.

## 5. Editor groups and focus

### 5.1 Group layout and resizing

- Split controls provide the two supported orientations; the maximum remains **two groups**.
- The divider is a **thin line that is itself draggable**, modeled on VS Code, without a permanent grip or thick rail.
- A wider invisible hit target improves pointer usability. Hover shows a subtle highlight and the appropriate resize cursor.
- For side-by-side groups, drag the vertical divider; for stacked groups, drag the horizontal divider.
- Double-click resets group proportions to **50/50**. Provide keyboard-accessible resizing and enforce minimum usable group dimensions.
- Save user-selected orientation and proportions; constrained effective geometry must not overwrite the preferred layout.

### 5.2 Active group indicator — WA-07 finalized

- The active group has a **subtle accent on its header** plus a slightly stronger header background.
- Do not draw a prominent border around the entire group.
- Active-group styling is distinct from active-tab styling, keyboard focus rings, and selected cells/records within an editor.
- Pointer or keyboard interaction that establishes editor focus updates the active group. Contextual commands and appropriate open/navigation actions target that group.
- The other group remains fully readable; switching focus must not change its active tab or editor-local state.

### 5.3 Group action placement

- At the **right end of each group's tab bar**, display a visible split control and an overflow menu for less frequent group actions.
- Overflow can include orientation changes, close-group actions, and other supported group-management commands.
- When width is constrained, secondary controls may collapse into overflow while keeping group management discoverable.
- Actions must respect the two-group ceiling and safe close behavior.

## 6. Dragging tabs

The drag destination determines both feedback and operation:

| Pointer destination                 | Feedback                                  | Drop result                              |
| ----------------------------------- | ----------------------------------------- | ---------------------------------------- |
| Existing group's **editor content** | Highlight destination work area           | Move tab into that group                 |
| Existing group's **tab strip**      | Accent insertion line at precise position | Insert/reorder tab at indicated position |
| Eligible **editor edge**            | Highlight proposed new split region       | Create second group and place tab there  |

- Edge splitting is only available when it would not exceed two groups.
- Ineligible drop targets must not suggest an action that cannot complete.
- Cancelling or leaving a valid target clears transient highlights without changing tabs.
- Preserve preview/permanent/pinned semantics and safe editor state when moving tabs.
- Pointer drag-and-drop is supplemented by keyboard-accessible commands for moving/reordering tabs and splitting groups.
- The UI uses shared commands and existing tab identity rules; dragging never duplicates canonical resources.

## 7. Breadcrumb navigation and overflow

### 7.1 Smart segment behavior

- Breadcrumbs display the current editor resource's meaningful ancestry, with the current resource clearly distinguished.
- **Click an openable ancestor:** navigate directly using established resource-opening rules.
- **Click a non-openable ancestor:** open that segment's contextual menu.
- **Click a Table ancestor:** activate its last-used valid View, falling back to its default View.
- The current resource need not navigate to itself; its context actions remain available.
- **Right-click any segment:** show the contextual menu. Keyboard users can activate or open context actions through standard keyboard conventions.
- Explicit **Reveal in Explorer** may open/reveal the Left Panel. Ordinary breadcrumb navigation or selection does not automatically reopen a closed panel or switch away from Search.
- Opening from breadcrumbs respects tab reuse, preview/permanent behavior, and group context.

### 7.2 Long paths

- Preserve the **current resource** and the most useful trailing ancestors whenever possible.
- Truncate the **middle** of long paths, replacing hidden segments with a **single clickable ellipsis (`…`)**.
- **No separate dropdown chevron** appears beside that ellipsis.
- Clicking the ellipsis opens a menu of hidden ancestors in hierarchy order; selecting an ancestor follows the same smart navigation/context-menu rules.
- Recompute visible segments as group width changes, including when split editors are resized. Do not replace the breadcrumb with horizontal scrolling.

Example:

```text
Workbench  ›  …  ›  Revenue  ›  Summary
                └─ Finance
                   North America
                   Quarterly Reporting
```

## 8. Resource toolbar

- The resource-specific toolbar sits **directly beneath the breadcrumb**, within the relevant editor group.
- Common, high-priority actions are visible; resource-specific commands and advanced actions are available through contextual controls and overflow.
- As width decreases, toolbar controls respond in this order:
  1. Show icons **and labels** where space allows.
  2. Collapse eligible labels to **icon-only** controls.
  3. Move **lower-priority** controls into the ellipsis overflow menu.
- Keep essential commands visible as long as practical; preserve keyboard access, tooltips, and accessible names for icon-only actions.
- Toolbar priorities are defined by the editor type; no single fixed action set is imposed on all editors.
- Frequent View controls can open popovers; advanced View/Field configuration continues to use the contextual Right Panel or other previously approved surfaces.
- This is a **Work Area placement/responsiveness rule**, not a redesign of Grid, Query, Dashboard, File, or Automation editing commands.

## 9. Empty editor group

Use the approved **compact launch surface**, not a full-screen welcome experience.

- Show **New Resource** and **Quick Open** as the primary actions.
- Show up to **five recently accessed resources**, with resource icons and disambiguated labels, when history is available.
- New Resource honors the approved context-sensitive destination rule: a selected Collection or resource within it establishes the default Collection destination; otherwise use Workbench root, subject to resource-placement restrictions (e.g., Files root-only and Views Table-owned).
- Recent-resource selection follows normal opening/activation behavior in the target group.
- Under limited width/height, prioritize the two actions and collapse or hide recent entries.
- With no recents, omit the recent-resources section entirely.
- The launch surface occupies **only the empty group**; it never replaces a populated neighboring group.
- Opening a resource replaces the launch surface without disturbing unrelated tabs/groups.

## 10. Loading, failures, and unavailable resources

Use **B+ contextual feedback with full-editor fallback**.

| Situation                                                                 | Treatment                                                                                          |
| ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Initial editor/resource loading                                           | Lightweight editor-local loading state                                                             |
| Partial load, refresh, or operation failure                               | Inline feedback near affected content, with applicable retry/recovery                              |
| Editor remains partially usable                                           | Preserve usable content and unaffected controls                                                    |
| Resource cannot be opened (deleted, inaccessible, missing, irrecoverable) | Full-editor explanation and appropriate recovery/navigation actions                                |
| Ongoing background work, Problems, History                                | Use existing Operations/Problems/History surfaces as appropriate; do not conflate with dirty state |

- Avoid replacing an entire editor for localized failures.
- Explain the condition clearly and offer actionable recovery only when supported.
- Preserve other editors and groups; a failure in one group must not unnecessarily block the other.
- Distinguish resource-unavailable states from stale/fresh derived data and from pending canonical writes.
- Error and recovery controls remain keyboard-accessible and use the shared command system.

## 11. Persistence, state, and commands

- Restore tabs, groups, active resources, orientation, proportions, and meaningful editor-local selection/scroll/drafts safely, consistent with existing V1 session restoration.
- Maintain distinct **window/group presentation state** and **canonical Core resource state**.
- Contextual commands follow the active editor/group and are enabled only where valid.
- Resource names/paths are presentation; use UUID-backed identity for navigation and tab reuse.
- Window-local focus and layout may differ across windows sharing a single Workbench Core; never create independent canonical writers.
- History/Back/Forward, Quick Open, Search, Explorer, and contextual reveal remain separate but interoperable navigation mechanisms.
- All editor-level interactions should preserve keyboard-only access, meaningful focus restoration, screen-reader descriptions, and applicable WCAG 2.2 AA targets.

## 12. Boundary with other detailed-design documents

**ShellDesign.md** owns overall shell placement, panel visibility, layout presets, responsive panel priorities, title/menu/Command Center, and shared shell chrome.  
**LeftPanelDesign.md** owns Explorer, Search, Activity Bar, and their selection/reveal behaviors.  
**RightPanelDesign.md** will own contextual inspection, Field/Record/Note editing, and advanced configuration panel patterns.  
**BottomPanelDesign.md** will own Problems, Operations, and History panel layout and interaction.  
**Resource-specific designs** will own Grid, Kanban, Calendar, Gallery, Pivot, Query, Dashboard, File, Automation, and tool editor interiors. This document only standardizes their Work Area host, headers, tabs, grouping, and common feedback.

## 13. Integrated visual-review outcome

The integrated Work Area mockup was reviewed and its overall layout **approved**. WA-07 was explicitly finalized as **Option B: subtle active-group header accent**, with a slightly stronger header background and no full-group outline. The review did not introduce additional functionality or change previously approved shell/resource semantics.

**Work Area detailed design status: complete.** The next detailed-design phase is **Right Panel Design**. No GitHub repository files were modified by preparation of this document.
