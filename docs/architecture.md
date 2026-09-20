# Workbench Architecture

Status: Working draft

## Purpose

This document records the technical architecture of Workbench, including confirmed decisions, their rationale, and questions requiring further evaluation.

The requirements document defines what Workbench must do. This document records how we have chosen to build it.

## Document Conventions

- Confirmed: An agreed architectural decision.
- Proposed: A recommended approach awaiting agreement.
- Tentative: A possible approach whose inclusion remains undecided.
- Open: A question requiring a decision.

A proposed or tentative approach is not a confirmed decision.

## 1. Application Platform

### Confirmed

- ARCH-01: Workbench V1 is a standalone desktop application built with Electron.
- ARCH-02: The desktop application supports Windows, macOS, and Linux.
- ARCH-03: The desktop interface uses React and TypeScript.
- ARCH-04: Desktop-specific capabilities remain separate from reusable application logic.

### Rationale

Electron provides a cross-platform desktop shell using web technologies. React provides flexibility for Workbench's spreadsheets, record details, dashboards, and multi-pane work area.

### Open

- Responsibilities of Electron's main process, renderer, and any workers.
- Desktop packaging, distribution, and update implementation.

## 2. Repository and Shared Core

### Confirmed

- ARCH-05: Workbench uses a monorepo containing the desktop application and reusable packages.
- ARCH-06: The reusable core is framework-independent TypeScript.
- ARCH-07: Data models, validation, formulas, and sync logic do not depend on React or Electron.
- ARCH-08: The architecture accommodates a future mobile companion without including mobile development in V1.

### Rationale

A monorepo allows coordinated development while preserving boundaries between the desktop application and shared code. A framework-independent core can be reused by future clients.

### Proposed

- Organize the repository around `apps/desktop` and `packages/core`.

### Open

- Final package structure, names, and dependency boundaries.
- Build tooling and package management.

## 3. Local Data Storage

### Confirmed

- ARCH-09: SQLite is Workbench's authoritative local database.
- ARCH-10: Attachment content is stored as separate files, with metadata and references tracked in SQLite.
- ARCH-11: Core editing works offline without requiring a server connection.

### Rationale

SQLite supports transactions, indexed queries, and updates across related data without a database license or required database server. Separate attachment files avoid placing all file content directly into database records.

### Open

- One SQLite database per workbench versus one database for the application.
- Database schema and migration strategy.
- Attachment file layout and recovery procedures.
- Full-workbench backup and restore format.

## 4. Spreadsheet Grid

### Confirmed

- ARCH-12: Prioritize free, commercially usable grid libraries.

### Rationale

Workbench's core spreadsheet experience must not depend on paid-only library features. The grid must support the spreadsheet interactions defined in the requirements.

### Open

- Which grid library or rendering foundation to use.
- How much spreadsheet behavior Workbench should implement itself.
- How to keep dataset and editing logic independent of the chosen grid.
- How to validate performance, keyboard interaction, and accessibility against the requirements.

## 5. Synchronization

### Confirmed

- ARCH-13: Cross-device synchronization is part of V1.
- ARCH-14: Local editing must remain available without a network connection.
- ARCH-15: A hosted service's free tier is acceptable, but a paid plan must not be required for the intended personal-use experience.
- ARCH-16: Full-workbench backup and export must not depend on the sync provider.

### Rationale

Synchronization should complement local-first operation rather than become a prerequisite for using Workbench. Backup and export must remain available independently of the selected service.

### Open

- Sync provider or transport.
- Representation and transfer of changes and attachments.
- Conflict handling and recovery implementation.
- Free-tier limits, commercial-use terms, and behavior when quotas are reached.

## 6. Costs and Dependencies

### Confirmed

- ARCH-17: Prioritize free, commercially usable software dependencies.
- ARCH-18: Workbench V1 must be possible to build and personally use without required ongoing payments.

### Open

- Which optional paid services or components, if any, may be supported without becoming V1 requirements.
- How to handle changes to third-party free-tier terms or limits.

## 7. Architectural Principles

### Confirmed

- ARCH-19: Keep the user interface, reusable business logic, local persistence, and synchronization responsibilities separate.
- ARCH-20: Preserve the data portability and recovery behaviors defined in the requirements.
- ARCH-21: Avoid desktop-only dependencies in code intended for reuse by a future mobile client.

These principles guide later decisions but do not prescribe implementations that have not yet been evaluated.

## 8. Remaining Architecture Decisions

### Open

- SQLite database layout.
- Grid foundation and ownership.
- Sync provider and protocol.
- Electron process boundaries.
- Data schema, references, formulas, and migrations.
- Attachment organization, backup, and restore.

Confirmed decisions will move into their relevant sections as they are made. This document will be updated without changing the requirements unless a product requirement is explicitly revised.
