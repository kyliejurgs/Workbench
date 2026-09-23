# Workbench Coding Standards

Status: Working draft

## Purpose

This document defines coding conventions and implementation boundaries for Workbench.

The architecture document defines how the system is structured. These standards define how code within that architecture should be written and organized.

Standards should remain practical. New rules should be added when they solve a recurring problem or protect an important architectural boundary rather than attempting to anticipate every future implementation detail.

## 1. General Principles

- Prefer clear, maintainable code over clever or unnecessarily abstract code.
- Keep responsibilities narrow and explicit.
- Avoid premature abstraction. Extract shared behavior when a meaningful reusable concept exists.
- Prefer composition over large multipurpose classes, components, or services.
- Business rules should remain independent of UI and infrastructure whenever practical.
- Avoid hidden side effects.
- Do not duplicate authoritative state unnecessarily.
- Architectural boundaries defined in `architecture.md` must not be bypassed for convenience.
- New dependencies should solve a meaningful problem and should not duplicate capabilities already provided by the project.

## 2. Repository and File Organization

Workbench uses npm workspaces.

The repository begins with the following high-level structure:

    apps/
      desktop/

    packages/
      core/

    docs/

Desktop-specific code belongs under `apps/desktop`.

Framework-independent business logic belongs under `packages/core`.

Additional packages should be created only when a meaningful architectural boundary exists. Do not create packages solely to reduce directory size.

Within the desktop renderer, organize application code primarily by feature rather than technical type.

Example:

    src/
      app/
      features/
        explorer/
        work-area/
        tables/
        settings/
      components/
      hooks/
      services/
      styles/

Feature-specific components, hooks, types, and utilities should remain with their feature.

The shared `components` directory is reserved for genuinely reusable UI components.

Avoid catch-all directories containing unrelated code.

## 3. TypeScript

TypeScript strict mode is required throughout the project.

Do not use `any` as an escape hatch from the type system.

When a value is genuinely unknown, use `unknown` and narrow it explicitly.

Prefer explicit domain types over loosely structured objects.

Public APIs, shared package boundaries, IPC contracts, and persisted data structures must be strongly typed.

Prefer discriminated unions when representing values with distinct states.

Avoid unnecessary type assertions.

Type assertions must not be used merely to silence compiler errors.

Prefer immutable inputs where mutation is not required.

Use `null` and `undefined` deliberately. Their meanings should not be interchangeable within a domain model.

Export types only when they are needed outside their module.

## 4. Naming

Use descriptive names that reflect domain meaning.

Use:

- `PascalCase` for components, classes, types, and interfaces.
- `camelCase` for variables, functions, hooks, and object properties.
- `UPPER_SNAKE_CASE` for true constants where appropriate.
- `kebab-case` for directories and non-component filenames.
- `PascalCase` for React component filenames.

Boolean names should normally describe a true condition:

    isSelected
    hasChanges
    canEdit
    shouldSync

Avoid vague names such as:

    data
    info
    item
    thing
    obj
    temp

unless the surrounding context makes their meaning unambiguous.

Domain terminology should match the terminology established in the requirements and architecture documents.

For example, use `workbench`, `table`, `record`, `field`, `view`, `query`, `dashboard`, `collection`, and `file` consistently rather than introducing alternate names for the same concepts.

## 5. Imports and Module Boundaries

Use configured path aliases for stable project boundaries.

Examples:

    import { Workbench } from '@workbench/core';
    import { Button } from '@/components/Button';

Avoid long relative import chains such as:

    ../../../../components/Button

Relative imports are acceptable for nearby files within the same feature or module.

Do not import another feature's internal implementation directly.

If functionality must be shared between features, expose an intentional public interface or move the shared concept to an appropriate shared location.

Circular module dependencies are not permitted.

## 6. React and UI

Use functional React components.

Components should have a clear responsibility.

Avoid components that combine unrelated application behavior, data access, layout, and presentation.

Feature-specific components remain within their feature.

Shared components should be domain-neutral whenever practical.

Prefer composition over large collections of configuration flags.

Do not place business rules directly in presentation components when those rules can live in the core or feature logic.

Hooks should represent meaningful reusable behavior rather than simply moving code out of a component.

Do not use React context as a default state-management mechanism. Context should represent genuinely shared contextual state.

Avoid storing values in React state when they can be derived from existing state.

Effects should synchronize React with external systems. They should not be used as a general-purpose mechanism for deriving state.

UI code must preserve keyboard accessibility and visible focus behavior.

## 7. State and Data Flow

Maintain a clear distinction between:

- persisted domain state,
- application state,
- view state,
- temporary editing state.

SQLite-backed data is authoritative for persisted local workbench state.

React component state must not become an independent authoritative copy of persisted domain data.

Optimistic UI updates are permitted, but persisted changes are considered durable only after the underlying database transaction succeeds.

Derived values should be calculated from authoritative state rather than stored redundantly unless caching is intentional.

State ownership should be as close as practical to the code responsible for it.

Global state should be introduced only for genuinely application-wide concerns.

## 8. Electron and IPC

The renderer must not directly access:

- Node.js APIs,
- Electron privileged APIs,
- SQLite,
- the filesystem,
- operating-system APIs.

Privileged capabilities are exposed through the preload layer using an intentionally defined and typed API.

The preload API must expose application capabilities rather than unrestricted low-level primitives.

Prefer:

    workbench.open(...)
    table.create(...)
    file.select(...)

over APIs that expose arbitrary filesystem, database, or IPC access.

IPC contracts must be strongly typed.

IPC inputs must be validated at the privileged boundary.

Do not trust renderer-provided paths, identifiers, or payloads solely because they originated from Workbench UI code.

Electron main-process code coordinates application and window behavior.

Database work belongs to the designated data service rather than the renderer or UI layer.

Long-running or expensive operations must not unnecessarily block the renderer or Electron main process.

## 9. Core Package

`packages/core` contains framework-independent Workbench domain logic.

Core may contain concepts such as:

- workbenches,
- tables,
- fields,
- records,
- references,
- formulas,
- validation,
- queries,
- synchronization rules.

Core must not depend on:

- React,
- Electron,
- browser UI APIs,
- desktop window behavior,
- SQLite connection implementations.

Core logic should be usable by another client without requiring the Electron desktop application.

Domain behavior that can be expressed independently of storage or presentation should normally live in core.

Infrastructure-specific implementations should depend on core rather than core depending on infrastructure.

## 10. CSS and Theming

Application styling uses centralized design tokens.

Components must use semantic design tokens rather than hard-coded theme colors.

Prefer:

    background: var(--surface-secondary);
    color: var(--text-primary);
    border-color: var(--border-default);

Do not use arbitrary color values inside component styles when an appropriate token exists.

The initial style organization includes:

    styles/
      colors.css
      tokens.css
      globals.css

`colors.css` defines the available color primitives and theme-specific color values.

`tokens.css` defines semantic application tokens.

`globals.css` defines application-wide base styles and global behavior.

Light mode, dark mode, system appearance, and accent colors should be implemented by changing token values rather than maintaining separate component stylesheets for each theme.

Feature-specific styles should remain with their feature or component.

Avoid global selectors when a style applies only to one component or feature.

Do not use `!important` as a routine specificity solution.

## 11. Error Handling and Logging

Do not silently swallow unexpected errors.

Expected domain failures should be represented explicitly and handled at an appropriate boundary.

User-facing errors should explain what failed and, where practical, what the user can do next.

Internal error details should remain available for diagnostics without exposing inappropriate implementation details to the UI.

Catch errors where meaningful recovery, translation, logging, or user feedback can occur.

Avoid catch-and-rethrow code that adds no useful context.

Database and filesystem operations must handle expected failure conditions.

Errors crossing process boundaries must use a defined serializable representation.

Do not rely on `console.log` as the application's long-term logging strategy.

## 12. Testing

Testing should protect behavior and important architectural boundaries rather than target an arbitrary coverage percentage.

Domain logic should be testable independently of Electron and React.

Tests are expected for:

- business rules,
- validation behavior,
- formula and reference behavior,
- data transformations,
- synchronization and conflict-resolution logic,
- complex state transitions,
- database behavior where correctness depends on transactions or persistence,
- regressions for previously discovered bugs.

UI tests should focus on meaningful user behavior rather than implementation details.

A bug fix should include a regression test when the failure can reasonably be reproduced through automated testing.

Tests should be deterministic.

Do not make tests depend unnecessarily on execution order, network access, wall-clock timing, or shared mutable state.

## 13. Documentation and Comments

Prefer code that explains itself through structure and naming.

Comments should explain why something exists, an important constraint, or behavior that would otherwise be surprising.

Do not write comments that simply repeat the code.

Public interfaces and complex domain behavior should be documented when their contract is not obvious from their types.

TODO comments must describe a concrete remaining task.

Architecture or product decisions discovered during implementation should be reflected in the appropriate documentation rather than existing only in comments or commit history.

## 14. Formatting and Static Analysis

Workbench uses ESLint and Prettier.

Formatting is automated.

Code should not be manually formatted in ways that intentionally fight the configured formatter.

Lint rules should primarily protect correctness, consistency, and architectural quality.

Lint rules that create noise without providing meaningful value should not be retained merely for strictness.

Lint and type-check failures must be resolved before code is considered complete.

## 15. Evolving These Standards

These standards are expected to evolve as Workbench is implemented.

New standards should be added when implementation reveals a recurring decision, ambiguity, or failure mode.

Do not create rules solely for hypothetical future problems.

When an established standard no longer serves the application, update the standard rather than repeatedly creating exceptions.
