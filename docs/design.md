# Workbench Design

Status: Working draft

## Purpose

This document defines the primary visual structure and interaction model for the Workbench desktop application.

It complements `requirements.md` and `architecture.md`. Requirements define what the application does, architecture defines how it is built, and this document defines how users interact with it.

Workbench is a new application and does not reuse the Better Spreadsheet implementation. However, established interaction and design decisions from Better Spreadsheet are retained where they remain applicable.

## 1. Design Direction

Workbench is a dense, desktop-first productivity application.

The interface should prioritize usable workspace over decorative UI while remaining visually polished and understandable.

The general interaction model is inspired by desktop development and productivity tools such as VS Code rather than traditional document applications.

Primary principles:

- Maximize useful working space.
- Keep frequently used actions immediately accessible.
- Prefer persistent context over unnecessary navigation.
- Allow advanced functionality without making basic workflows complicated.
- Use consistent interaction patterns across content types.
- Avoid excessive cards, containers, padding, and decorative whitespace.
- Prefer subtle visual separation over heavy borders.
- Support efficient mouse and keyboard workflows.
- The interface should feel like a desktop application rather than a website.

## 2. Application Shell

Each application window displays one workbench.

The primary application structure consists of:

    ┌─────────────────────────────────────────────────────────┐
    │ Title / Window Area                                     │
    ├─────┬───────────────┬──────────────────────────┬────────┤
    │     │               │                          │        │
    │ A   │   Sidebar     │       Work Area          │ Right  │
    │ c   │               │                          │ Panel  │
    │ t   │               │                          │        │
    │ i   │               │                          │        │
    │ v   │               │                          │        │
    │ i   │               │                          │        │
    │ t   │               │                          │        │
    │ y   │               │                          │        │
    ├─────┴───────────────┴──────────────────────────┴────────┤
    │ Status Area                                             │
    └─────────────────────────────────────────────────────────┘

The Work Area is the primary application surface.

Supporting UI should consume only as much space as necessary.

Panels should generally be resizable and hideable.

## 3. Activity Area

A narrow activity area appears along the left edge of the application.

It provides access to major application contexts rather than individual pieces of workbench content.

Initial activities include:

- Explorer
- Search
- Files
- Automations
- Settings

The selected activity determines the content displayed in the adjacent sidebar.

Activity buttons use icons with tooltips.

The activity area remains narrow and visually subordinate to the Work Area.

## 4. Sidebar

The sidebar displays contextual navigation and tools for the selected activity.

It is resizable and can be collapsed.

### Explorer

The Explorer is the primary navigation interface for the active workbench.

It displays:

- Collections
- Ungrouped tables
- Saved queries
- Dashboards

Collections are non-nesting.

Tables, queries, and dashboards can be moved between collections or left ungrouped.

Files are managed through the Files activity rather than being mixed into organizational collections.

Explorer items support appropriate context menus and drag-and-drop organization.

Double-clicking or otherwise activating an openable item opens it in the Work Area.

The Explorer should remain compact enough to support workbenches containing many items.

## 5. Work Area

The Work Area is a host for open content.

Multiple content types can be open simultaneously.

Examples include:

- Tables
- Saved views
- Queries
- Dashboards
- Files
- Settings or specialized editors where appropriate

Open content appears in tabs.

Only the active tab in a tab group is displayed.

Opening another item does not replace already-open content.

The same underlying content may be opened in more than one tab when different views or contexts make that useful.

### Tab Groups

The Work Area supports multiple tab groups.

Groups can be arranged side by side.

Users can:

- create split groups,
- resize groups,
- move tabs between groups,
- close tabs,
- reorder tabs.

Each group has its own active tab.

The Work Area should remain content-type agnostic. Individual features provide the content rendered inside it.

## 6. Tables

Tables use a spreadsheet-style interface and are the primary data-editing surface.

The table interface should prioritize data density and efficient editing.

A table generally consists of:

    ┌───────────────────────────────────────────────────────┐
    │ Table / View Toolbar                                  │
    ├───────────────────────────────────────────────────────┤
    │ Field │ Field │ Field │ Field │ Field │ ...           │
    ├───────┼───────┼───────┼───────┼───────┼───────────────┤
    │       │       │       │       │       │               │
    │       │       │       │       │       │               │
    │                    Records                            │
    │                                                       │
    └───────────────────────────────────────────────────────┘

Spreadsheet behavior should feel familiar to users of Excel and Google Sheets without attempting to duplicate either application exactly.

The interface supports:

- single-cell selection,
- range selection,
- keyboard navigation,
- direct cell editing,
- copy and paste,
- fill behavior,
- row operations,
- column resizing,
- column reordering,
- multi-cell operations.

Field headers provide access to field configuration and relevant sorting, filtering, and other field actions.

Invalid values remain visible and are visually distinguishable without overwhelming the grid.

Formula overrides, broken references, synchronization state, and other exceptional states should use consistent indicators.

## 7. Views

A table may have multiple saved views.

Views change presentation rather than duplicating records.

The active view should be visible near the table's primary controls.

V1 view types include:

- Spreadsheet
- Cards

Switching views should not feel like opening an unrelated dataset.

View-specific controls appear only when relevant to the active view.

Filters, sorting, grouping, visible fields, and presentation configuration belong to the view.

## 8. Record Details

Selecting or opening a record can display its details in a right-side panel.

The panel provides access to information that is inconvenient to display directly in the primary view.

It may include:

- all fields,
- references,
- attachments,
- notes,
- comments.

The panel follows the current record selection by default.

Users can pin a record to keep its details visible while navigating elsewhere.

The panel is resizable and hideable.

Editing in the panel changes the same underlying record displayed by tables and views.

The panel should not unnecessarily replace the user's current Work Area content.

## 9. Files

Files are first-class workbench content but are managed through a central Files area.

The Files interface provides access to the workbench's file library.

Files should support appropriate list/grid presentation depending on content and future requirements.

Where supported, files can be previewed without leaving Workbench.

References to files from records or other content open or reveal the same underlying file rather than creating duplicate copies.

File synchronization/download state should be visible when relevant without dominating the normal interface.

## 10. Dashboards

Dashboards are primarily read-only presentation surfaces.

Dashboard layouts use configurable widgets.

Widgets include:

- charts,
- statistic cards,
- filtered tables.

Widgets can be arranged and resized.

Dashboard editing should be visually distinguishable from normal dashboard viewing.

Users can interact with widgets to reach the underlying records rather than editing source data directly inside dashboard visualizations.

## 11. Search

Search is available as a dedicated activity.

Search defaults to the active workbench.

Users can broaden search to all locally available workbenches.

Search results clearly identify:

- matching content,
- content type,
- source table or context,
- source workbench when applicable.

Opening a search result navigates directly to the relevant content or record.

Search UI should support rapid keyboard-driven navigation.

## 12. Toolbars and Commands

Toolbars should contain actions relevant to the currently displayed content.

Do not create large permanent toolbars containing actions that apply only occasionally.

Common actions may be represented by icons when their meaning is clear.

Less common actions belong in menus or context menus.

Icons require tooltips when their meaning may not be obvious.

Destructive actions must be visually and behaviorally distinguishable from normal actions.

Context menus should expose actions appropriate to the selected object or current location.

## 13. Dialogs and Popovers

Use dialogs for interactions requiring focused user attention or explicit confirmation.

Use popovers or menus for lightweight configuration and selection.

Avoid using modal dialogs for routine editing when inline editing, a panel, or a popover provides a better workflow.

Dialogs should not become miniature application screens unless the task genuinely requires it.

Destructive confirmations should explain what will happen rather than relying only on generic Yes/No language.

## 14. Drag and Drop

Drag and drop should be used where spatial movement is intuitive.

Examples include:

- reorganizing Explorer items,
- moving items between collections,
- moving tabs,
- rearranging dashboard widgets,
- reordering fields where supported.

Drag operations require clear destination feedback.

Drag and drop should not be the only way to perform an important action.

## 15. Visual System

Workbench uses a compact visual system appropriate for a data-heavy desktop application.

The design favors:

- restrained spacing,
- compact controls,
- subtle borders,
- clear hierarchy,
- limited decorative effects,
- consistent control heights,
- consistent icon sizing.

Rounded corners may be used where appropriate but should not make every surface appear as an independent card.

Large amounts of unused whitespace should be avoided in working interfaces.

The application should maintain clear visual hierarchy without sacrificing information density.

## 16. Color and Themes

Workbench supports:

- Light
- Dark
- Follow System

Users can select an accent color independently of the base theme.

Accent color is used for meaningful interactive emphasis such as:

- active selections,
- selected navigation,
- focus states,
- active controls,
- appropriate highlights.

Accent color should not recolor the entire interface.

Semantic colors represent states such as:

- error,
- warning,
- success,
- informational state.

Components consume semantic design tokens rather than hard-coded theme colors.

Theme changes should apply immediately without requiring an application restart.

## 17. Typography

Workbench uses a clean sans-serif interface typeface suitable for dense desktop UI.

Typography establishes hierarchy primarily through:

- size,
- weight,
- spacing,
- semantic color.

Avoid excessive variation in font sizes.

Data-heavy surfaces should prioritize readability and density.

Monospace typography may be used where the content benefits from fixed-width presentation, such as formulas, technical identifiers, or code-like values.

## 18. Icons

Use one consistent icon family throughout the application.

Icons should communicate common actions without unnecessary decoration.

Do not mix visually incompatible icon sets.

Icons used without visible labels require accessible names and appropriate tooltips.

## 19. Empty States

Empty states should help the user perform the next useful action.

For example, an empty workbench should guide users toward creating or importing their first table rather than displaying a large decorative illustration.

Empty states should remain concise and functional.

## 20. Feedback and Status

Workbench should clearly communicate meaningful background state without interrupting normal work.

Examples include:

- saving,
- synchronization,
- offline status,
- validation problems,
- file transfer,
- long-running operations,
- errors.

Routine successful autosaves should not generate distracting notifications.

Use persistent indicators when a state remains relevant and temporary notifications for completed or exceptional events.

## 21. Window Size and Layout

Workbench is desktop-first.

The interface does not need to transform into a mobile layout when the window becomes narrow.

At smaller supported window sizes:

- optional panels may collapse,
- sidebars may be hidden,
- toolbars may move lower-priority actions into overflow menus,
- the Work Area retains priority.

Data surfaces may scroll rather than compressing until they become unusable.

A minimum practical window size will be established during implementation.

## 22. Accessibility

All primary workflows must be usable without a mouse.

Interactive elements require visible keyboard focus.

Icon-only controls require accessible names.

Color must not be the only indicator of state.

Text and interface scaling must not make core workflows unusable.

Reduced-motion preferences should be respected.

Spreadsheet and other complex data surfaces should expose appropriate semantic information to assistive technologies.

## 23. Design Consistency

Equivalent actions should behave consistently across Workbench.

Shared interaction patterns should be implemented through shared UI primitives rather than independently recreated by each feature.

New UI patterns should be introduced only when existing patterns do not adequately support the interaction.

When implementation reveals a reusable interaction rule, update this document rather than allowing similar features to evolve independently.
