# Workbench Version 1 Specifications

**Purpose:** Authoritative definition of the intended Workbench v1 product and high-level architecture.

# 1. Product Definition

Workbench is a local-first desktop application for building, organizing, analyzing, and visualizing structured data.

It combines the directness and flexibility of a spreadsheet with the structure, relationships, and analytical capabilities of a database.

The central product principle is:

> **Feel like a spreadsheet; behave like structured data underneath.**

Structured data is the core of Workbench. The grid is a primary interface for working with that data, but the grid does not define the underlying data model.

Workbench is not intended to be a general-purpose document editor, programming IDE, notebook environment, or productivity suite. Features should serve the creation, management, analysis, or presentation of structured data.

## Terminology

**Workbench** refers to the desktop application.

A **workbench** is a self-contained unit of work containing its data, structure, relationships, queries, dashboards, files, history, and other supporting resources.

A workbench is represented to the user as a portable `.wkbn` artifact.

The terms **Workspace** and **Project** are not used as formal synonyms for a workbench.

---

# 2. Core Object Model

A workbench contains the following core resource types:

```text
Workbench
├── Collection
│   ├── Table
│   ├── Query
│   └── Dashboard
├── Table
│   ├── Field
│   ├── Record
│   └── View
├── Query
├── Dashboard
│   └── Widget
└── File
```

Tables, Queries, and Dashboards may exist either directly at the workbench level or inside a Collection.

Collections are optional and organizational. They cannot contain other Collections.

Files exist at the workbench level and are not organized into Collections.

## Table

A **Table** is the primary structured-data resource in Workbench. A Table owns its Fields, Records, and Views. Relationships and derived values associated with a Table are defined through its Fields.

## Field

A **Field** defines the meaning and type of a value associated with each Record in a Table. Fields have stable identities independent of their names, positions, or presentation.

## Record

A **Record** represents one structured entity within a Table. Records have stable identities independent of their current row positions or presentation.

## Value

A **value** exists at the intersection of a Record and a Field.

Conceptually:

```text
(recordId, fieldId) → value
```

A cell is a grid presentation of this intersection. It does not need to exist as an independent domain resource.

## View

A **View** is a saved presentation of a single Table. Views control how Table data is presented without owning or duplicating that data.

## Query

A **Query** is a saved, live, read-only derived dataset constructed from Tables and/or other Queries.

## Dashboard

A **Dashboard** is a saved presentation composed of Widgets that visualize or summarize Workbench data.

## Widget

A **Widget** is an individual presentation component within a Dashboard.

## Collection

A **Collection** optionally organizes Tables, Queries, and Dashboards. A Collection exists directly within a workbench and cannot contain another Collection. Moving a resource into or out of a Collection does not change its identity or dependencies.

## File

A **File** is a supporting resource stored at the workbench level and available to other Workbench resources where appropriate. Files are not contained within Collections.

---

# 3. Identity and Dependencies

Workbench resources and other persisted domain objects use stable internal identifiers.

A resource's identity is independent of its name, position, Collection, or presentation. Renaming, reordering, or moving a resource does not change its identity.

For example, the following are presentation or organizational properties and must not be used as persistent identity:

- Resource names
- Collection membership
- Field order
- Record position
- View position
- Explorer location

Dependencies reference stable identifiers rather than names or positions.

Conceptually:

```text
Formula
    ↓
Field ID

Reference
    ↓
Table ID + Record ID

Query
    ↓
Table / Query IDs

Dashboard Widget
    ↓
Table / Query ID
```

This allows resources to be renamed, reordered, or reorganized without breaking relationships or derived behavior.

## Broken Dependencies

Workbench preserves broken dependencies where practical rather than silently discarding them.

For example, if a referenced Record or source resource is deleted, dependent resources retain enough information to identify that the dependency is broken.

Broken dependencies are surfaced through Workbench's Problems system so the user can understand and resolve them.

## Dependency Awareness

Operations that may affect dependent resources should be dependency-aware.

Workbench should not silently cascade destructive changes through unrelated resources. When an operation would break existing dependencies, Workbench warns the user and identifies the affected resources where practical.

---

# 4. Workbench Boundary

A workbench is a self-contained unit of data, structure, analysis, presentation, and history. 

Resources and dependencies within a workbench may reference other resources in the same workbench.

This includes relationships between:

- Tables and Records
- Fields and References
- Formulas and their dependencies
- Queries and their sources
- Dashboards, Widgets, and their sources
- Attachments and Files
- History and recoverable state

Live dependencies do not cross workbench boundaries in v1. A resource in one workbench cannot directly reference a resource or Record in another workbench.

Data and resources may still move between workbenches through supported import, export, and native sharing mechanisms. Once imported into another workbench, they become part of that workbench and use local identities.

Cross-workbench live references or linked datasets may be considered in a future version.

---

# 5. Desktop and Window Model

Workbench is a multi-window desktop application.

Each application window contains either:

- one open workbench; or
- no open workbench, in which case the window displays the Home experience.

A single application window does not contain multiple workbenches simultaneously.

## Opening Workbenches

Opening a `.wkbn` artifact from the operating system opens that workbench in its own Workbench window.

When opening another workbench from within the application, Workbench allows the user to choose whether to open it in:

- **This Window**; or
- **New Window**.

Opening a workbench in the current window closes the current workbench presentation and replaces it with the newly opened workbench.

Because workbench data is autosaved, switching workbenches does not require a traditional unsaved-changes prompt.

## One Active Window per Workbench

The same workbench cannot be actively open in multiple Workbench windows at the same time. If the user attempts to open a workbench that is already open in another window, Workbench focuses the existing window instead of opening a second active instance.

## Application Launch

When Workbench is launched directly, it attempts to reopen the most recently opened workbench. If that workbench is unavailable or cannot be opened, Workbench displays the Home experience.

Workbench does not automatically recreate every application window that was open when the previous session ended.

## Closing Windows

Closing a Workbench window closes that application's presentation of the workbench. It does not require an explicit save operation under normal circumstances. Closing the final Workbench window exits the application.

---

# 6. Workbench Creation and Saving

Workbench uses a save-first creation model and automatic persistence.

## Creating a Workbench

Creating a new workbench requires the user to choose a name and storage location before the workbench is created. Workbench then creates the `.wkbn` artifact and opens it.

There is no normal untitled or unsaved workbench state.

## Autosave

Changes to a workbench are saved automatically.

Users should not need to manually save their work in order to preserve ordinary edits, configuration changes, or resource changes.

Closing a tab, switching workbenches, or closing the application does not normally require a save prompt.

Autosave must work together with Workbench's transactional persistence, history, and recovery systems so that automatic persistence does not compromise recoverability or data integrity.

## Manual Save Command

Workbench may provide the conventional **Save** command and `Ctrl/Cmd + S` keyboard shortcut.

Because ordinary changes are already persisted automatically, this command does not represent the primary mechanism for saving user work. Instead, it may explicitly flush pending persistence work and create or advance an appropriate durable checkpoint.

The exact checkpoint behavior is an implementation and history-design decision.

---

# 7. Application Shell

Workbench uses a desktop application shell designed for working with multiple structured-data resources within a workbench.

The shell provides persistent navigation, resource management, work surfaces, contextual tools, and application status without requiring each resource type to implement its own application-level interface.

Conceptually, the shell contains:

```text
Workbench Window
├── Title / Menu Area
├── Command Interface
├── Activity Bar
├── Left Panel
├── Work Area
│   └── Tabs
├── Bottom Panel
├── Right Panel
└── Status Bar
```

## Title and Menu Area

The title and menu area provides application-level commands and identifies the currently open workbench. Platform-native conventions should be respected where appropriate.

## Command Interface

Workbench provides a command interface for discovering and executing actions without requiring users to navigate menus or toolbars. The exact interaction model may evolve during UX design.

## Activity Bar

The Activity Bar provides access to major application-level views and tools, such as Explorer, Search, and/or History.

## Left Panel

The Left Panel displays the currently selected navigation view.

Explorer is the primary Left Panel view and provides access to the resources contained in the workbench. Other views, such as Search, may use the same panel.

## Work Area

The Work Area is the primary content surface. Views, Queries, Dashboards, Files, and other openable resources are presented here using tabs.

## Bottom Panel

The Bottom Panel provides supporting information and tools that benefit from remaining available alongside the active work surface. The Problems interface is presented here. Additional tools may use the Bottom Panel where appropriate.

## Right Panel

The Right Panel is available for contextual information, configuration, or inspection related to the active resource or selection. Its contents depend on context and it does not need to remain visible at all times.

## Status Bar

The Status Bar communicates persistent workbench and application status.

It may expose information and actions such as:

- Problems;
- background operation progress;
- current selection or resource context;
- other persistent status relevant to the active workbench.

Transient confirmations and ordinary notifications should not be treated as persistent status.

---

# 8. Tabs and Resource Opening

The Work Area uses tabs to present open Views and other openable resources. Tabs represent a particular resource or presentation rather than an entire Table.

## Tables and Views

A Table does not open directly into a generic Table tab. Instead, a Table is opened through one of its Views. Each open View has its own tab.

When the user opens a Table without specifying a View, Workbench opens:

1. the Table's last-used View, when available; or
2. the Table's default View.

Every Table has a default View.

Queries, Dashboards, and Files generally use one tab per open resource in v1.

## Preview Tabs

Workbench supports preview tabs for lightweight navigation.

Opening a resource through a preview action displays it in a reusable preview tab rather than immediately creating a permanent tab. Opening another resource through preview replaces the current preview tab unless that tab has been converted into a normal tab.

A preview tab becomes a normal tab when the user performs an action that demonstrates intent to keep it open, including:

- editing the resource;
- double-clicking the tab;
- explicitly choosing to keep the tab open.

If a resource is already open in a normal tab, Workbench activates that tab instead of opening another preview of the same resource.

Preview tabs may be used by Explorer, Search, Problems, and other navigation features.

## Closing Tabs

Closing a tab closes only that presentation in the Work Area. It does not delete the underlying resource or data.

Because Workbench uses autosave, closing a tab does not normally require a save prompt.

## Tab Restoration

Workbench stores enough presentation state to restore the user's working context when the workbench is reopened.

This includes:

- open tabs;
- tab order;
- active tab.

Restoring tabs does not change the underlying resources or create duplicate resources.

---

# 9. Explorer

Explorer is the primary interface for navigating and organizing resources within a workbench. It reflects the organizational hierarchy of the workbench without defining resource identity or dependency relationships.

## Structure

Explorer presents Collections and uncollected resources at the workbench level.

Collections may contain:

- Tables;
- Queries;
- Dashboards.

Collections cannot contain other Collections.

Tables expose their Views as child items.

Files are presented in a separate top-level **Files** section and are not contained within Collections.

Conceptually:

```text
Explorer
├── Collection
│   ├── Table
│   │   └── View
│   ├── Query
│   └── Dashboard
├── Table
│   └── View
├── Query
├── Dashboard
└── Files
    └── File
```

## Collections

Single-clicking a Collection expands or collapses it.

Collections are organizational containers and do not open in the Work Area. 

A disclosure control provides an explicit way to expand or collapse a Collection.

## Tables

Single-clicking a Table expands or collapses its Views. Double-clicking a Table opens its last-used View or, when unavailable, its default View.

A disclosure control provides an explicit way to expand or collapse the Table without opening it.

## Views and Other Openable Resources

Single-clicking a View, Query, Dashboard, or File opens it in preview. Double-clicking it opens it as a normal tab. If the resource is already open, Workbench activates the existing tab where appropriate.

## Ordering

Explorer uses deterministic ordering rather than manual item ordering.

At the workbench root:

1. Collections appear before uncollected resources.
2. Collections are ordered alphabetically.
3. Uncollected resources are ordered by resource type and then alphabetically within each type.

Within a Collection, resources are ordered by resource type and then alphabetically within each type.

Within a Table:

1. the default View appears first;
2. remaining Views appear alphabetically.

Workbench does not require visible headings between resource-type groups.

## Files

The Files section is flat in v1. Files can be filtered by filename, file type, or date added to make larger File collections easier to navigate. Additional filtering may be added later.

---

# 10. Naming

Workbench uses human-readable names for resources while maintaining stable internal identifiers separately. Names are validated within the scope in which the resource exists.

## Resource Names

Resources of the same type under the same parent must have unique names. Name uniqueness is case-insensitive. 

For example, the following resources may coexist within the same Collection:

```text
Customers    Table
Customers    Query
Customers    Dashboard
```

The following may not:

```text
Customers    Table
customers    Table
```

because they are two Tables with the same case-insensitive name under the same parent.

Moving a resource into a different parent changes the naming scope in which its name must be unique.

## Collections

Collection names must be unique among Collections in the workbench. Because Collections cannot be nested, all Collection names share the same naming scope.

## Views

View names must be unique within their Table. Different Tables may contain Views with the same name.

## Fields

Field names must be unique within their Table. Different Tables may contain Fields with the same name.

## Choice Options

Choice option names must be unique within their Field. This uniqueness is case-insensitive.

## Renaming

Renaming a resource changes its human-readable name but does not change its stable identity.

References, Formulas, Queries, Dashboards, and other dependencies must continue to function after a referenced resource is renamed.

Where appropriate, Workbench supports inline renaming in Explorer and other resource-management interfaces. Name validation occurs immediately enough to prevent the user from committing an invalid duplicate name.

---

# 11. Tables and Grid Philosophy

The Grid provides a spreadsheet-like interface for working directly with structured Table data. It should feel familiar to spreadsheet users while preserving the underlying Workbench data model.

A Grid presents real:

- Fields
- Records
- Values

The Grid does not define the identity or meaning of the underlying data.

## Structured Grid

Workbench does not use an infinite arbitrary cell canvas. Every persisted value belongs to a Record and a Field:

```text
(recordId, fieldId) → value
```

Creating data through the Grid therefore creates or modifies actual Table structure and Records rather than independent spreadsheet coordinates.

## Row Numbers

The Grid displays row numbers as positional navigation aids.

Row numbers reflect the current presentation of Records and may change as data is sorted, filtered, grouped, inserted, or removed. A row number is not a Record identifier and cannot be used as a persistent Reference or Formula dependency.

## Fields Instead of Column Letters

Workbench does not use spreadsheet column letters such as `A`, `B`, or `C` as Field identities. Columns represent named Fields.

Formulas, References, Queries, and other dependencies use Fields and their stable identities rather than spreadsheet coordinates.

## Empty Tables

A newly created Empty Table contains no Fields or Records. Workbench does not create placeholder Fields such as `Field 1` merely to make the Grid appear populated. Instead, the empty Grid provides an affordance for creating the first Field.

## Virtual Blank Rows

A Grid View may fill otherwise unused vertical viewport space with virtual blank rows. Virtual blank rows are presentation affordances only. They are not persisted Records.

Entering or pasting data into a virtual blank row creates a real Record as needed.

Virtual blank rows fill available viewport space rather than creating a large artificial scrollable dataset.

## Horizontal Space

Workbench does not create virtual blank Fields to fill unused horizontal Grid space. New Fields are created explicitly through Grid controls or other Field-management interfaces.

## Spreadsheet Interaction

Where compatible with the structured-data model, the Grid should support familiar spreadsheet interactions, including:

- keyboard navigation
- range and cell selection
- direct editing
- copy and paste
- multi-cell operations
- efficient data entry

Spreadsheet familiarity should improve interaction without introducing coordinate-based data semantics.

---

# 12. Table and Field Creation

Workbench provides multiple ways to create a Table while preserving the same underlying Table model.

## Table Creation

V1 supports three primary Table creation paths:

1. **Empty Table**
2. **Configure Table**
3. **Import**

Every Table requires a name before it is created.

### Empty Table

Empty Table creates a Table with no Fields or Records. The Table opens in its default Grid View, where the user can begin defining Fields or entering data.

### Configure Table

Configure Table allows the user to define initial Table structure before creation. This may include Fields, Field types, and other appropriate initial schema configuration.

The configuration experience should remain focused on initial structure rather than exposing every advanced Table or Field setting.

### Import

Import creates a new Table from external structured data. Detailed import behavior is defined separately in the Import section.

## Field Creation

Fields can be created directly while working in the Grid. The Grid provides an **Add Field** affordance at the right edge of the existing Fields.

Creating a Field requires a name. Unless another type is explicitly selected, a new Field uses the **Text** type. Field names must satisfy the Table's naming rules before creation is completed.

Advanced Field configuration should remain available without making basic Field creation unnecessarily complex.

## Creating Structure Through Paste

Pasting tabular data into an Empty Table may bootstrap its initial structure. Workbench may use the pasted data to suggest Fields, Field names, and Field types before creating the resulting structure and Records.

Type inference assists the user but should not silently redefine established Field types after the schema has been created.

---

# 13. V1 Field Types

Workbench v1 supports a broad set of semantic Field types. Field types define how values are stored, validated, edited, displayed, and used by other Workbench features.

Different user-facing Field types may share underlying value representations while preserving distinct semantics and behavior.

## Text

### Text

Stores plain text intended primarily for short or general-purpose values.

### Long Text

Stores plain text with editing and presentation behavior suited to multiline or longer content. Text and Long Text do not differ merely by an arbitrary character limit.

### Email

Stores text representing an email address.

Workbench provides email-specific validation and may provide appropriate actions for valid values. Invalid input is preserved and surfaced as a Problem rather than silently discarded.

### Phone

Stores text representing a telephone number.

Workbench provides phone-specific validation and may provide appropriate actions for valid values.

### URL

Stores text representing a URL.

Workbench provides URL-specific validation and may provide appropriate actions for valid values.

## Numeric

### Number

Stores general numeric values.

### Currency

Stores monetary numeric values.

Currency configuration belongs to the Field rather than individual values. A Currency Field represents one configured currency. Datasets requiring multiple currencies should model those currencies explicitly rather than mixing currency semantics within one Field.

### Percentage

Stores a numeric proportion.

Percentage values use fractional semantics internally. For example:

```text
12.5% → 0.125
```

Workbench provides human-friendly percentage entry and display.

### Rating

Stores a numeric rating with Field-level rating semantics and presentation. The supported range is configurable. A `1–5` range is an appropriate default.

## Boolean

### Boolean

Stores a boolean value.

A Boolean Field may support Field-level display styles such as:

```text
True / False
1 / 0
```

### Checkbox

Stores boolean semantics while providing checkbox-specific editing and presentation.

Boolean and Checkbox are separate user-facing Field types even when they share an underlying value representation. Blank remains distinct from `false`.

## Temporal

### Date

Stores a calendar date without a time-of-day or instant-in-time requirement.

### Date & Time

Stores an actual instant in time. Workbench supports timezone-aware interpretation and display.

### Time

Stores a wall-clock time independent of a calendar date and timezone.

### Duration

Stores an elapsed amount of time.

## Choice

### Choice

Stores zero or one option from a Field-defined set of choices.

### Multi-Choice

Stores zero or more options from a Field-defined set of choices. Choice options have stable identities independent of their names.

Options may include presentation metadata such as color. Choice option names are case-insensitively unique within their Field. Renaming an option does not change its identity.

## Relationships and Derived Values

### Reference

Stores zero or one relationship to a Record in another specified Table.

### Multi-Reference

Stores zero or more relationships to Records in another specified Table.

### Summary

Derives an aggregate value through a multi-record relationship.

### Formula

Derives a value from other Fields and relationships using a Formula expression.

## Files and Structured Content

### Attachment

Stores references to zero or more Workbench File resources.

### JSON

Stores validated structured JSON data.

Nested JSON properties are not themselves Workbench Fields in v1.

## Generated

### Generated

Stores a system-owned generated value.

V1 Generated modes include:

- Sequence
- Created Time
- Modified Time

Additional generated modes may be introduced later.

## Extensibility

Workbench should model Field types as well-defined type implementations rather than scattering type-specific behavior throughout the application. This does not imply a third-party Field type plugin system in v1. A dedicated geographic or location Field type is not included in v1.

---

# 14. References and Relationships

References are Workbench's primary mechanism for defining relationships between Records. A Reference Field targets exactly one Table.

## Reference

A **Reference** contains zero or one target Record.

Conceptually:

```text
Reference Field
    ↓
Target Table
    ↓
Target Record
```

## Multi-Reference

A **Multi-Reference** contains zero or more target Records from the same target Table. A single Reference or Multi-Reference Field does not target multiple Tables in v1.

## Stable Identity

References store relationships using stable Record identity rather than displayed values, names, or row positions. Changing the value used to display a referenced Record does not change the underlying relationship.

## Display Field

Each Reference Field specifies how referenced Records are presented to the user.

Workbench may suggest an appropriate display Field using signals such as:

1. A target Field matching or closely resembling the Reference Field's name
2. An obvious name-like or identifying Field
3. A preferred display Field configured for the target Table, when available
4. Another sensible displayable Field.

The user can confirm or change the selected display Field. Different Reference Fields targeting the same Table may use different display Fields.

## Following a Reference

A derived Reference may follow an existing relationship to expose a specific Field from the referenced Record.

For example:

```text
Customer
    → Reference to Customers

Customer Address
    → Customer.Address

Customer Email
    → Customer.Email
```

This preserves the relationship to the Customer rather than requiring the user to independently select the same Customer-related values multiple times.

## Editing Referenced Values

A value derived through a Reference cannot be overwritten from the referencing location. To change the source value, the user edits the source Record.

Workbench should provide convenient navigation to that source where appropriate.

## Live Relationships

References are live. Changes to source Records propagate to dependent Workbench behavior, including where applicable:

- Displayed Reference values
- Derived Reference Fields
- Formulas
- Filters
- Sorts
- Summaries
- Queries
- Dashboards

## Broken References

If a referenced Record becomes unavailable, Workbench preserves enough relationship information to identify the broken Reference. The value is not silently converted into an unrelated blank. Broken References are surfaced through the Problems system.

## Future Relationship Types

Polymorphic References capable of targeting Records from multiple Tables are not supported in v1. The architecture should avoid unnecessary assumptions that would make richer relationship models prohibitively difficult to introduce later.

---

# 15. Summary Fields

A **Summary** is a read-only derived Field that aggregates values across Records reached through a multi-record relationship.

For example:

```text
Projects
├── Tasks
│   └── Multi-Reference → Tasks
└── Total Hours
    └── Summary → Tasks.Hours → Sum
```

In this example, each Project may reference multiple Tasks, and `Total Hours` derives the sum of the `Hours` Field across those related Tasks.

## Summary Definition

A Summary identifies:

1. Multi-record relationship to follow
2. Field or Records to summarize
3. Aggregation operation

Supported operations depend on the type of data being summarized. V1 operations should include appropriate operations such as:

- Count
- Sum
- Average
- Minimum
- Maximum

Not every operation is valid for every Field type.

## Live Calculation

Summary values are live. When the related Records, relationships, or summarized values change, dependent Summary values update accordingly.

## Read-Only Values

Summary values are derived and cannot be directly overwritten. To change a Summary result, the user changes the underlying Records, relationships, or Summary definition. Where appropriate, Workbench may allow a Summary Field to be explicitly converted to ordinary stored values through **Convert to values**.

---

# 16. Formula Fields

A **Formula** is a read-only derived Field whose value is calculated from other Fields and relationships. Formula expressions begin with `=`.

## Field References

Formula expressions reference Fields by name using bracket syntax.

For example:

```text
=[Quantity] * [Unit Price]
```

Field names are used for human-readable formula presentation, while Workbench internally binds Formula dependencies to stable Field identities. Renaming a referenced Field therefore updates the displayed Formula without breaking its dependency.

## Relationship Traversal

Formulas may follow References to values on related Records.

For example:

```text
=[Customer].[Discount Rate] * [Subtotal]
```

This accesses the `Discount Rate` Field on the Record referenced by `Customer`.

## Formula Scope

Formulas operate on the current Record and its relationships.

Workbench does not use arbitrary spreadsheet-coordinate dependencies such as:

```text
=B17 * C17
=SUM(B2:B500)
```

The conceptual division is:

```text
Formula → current Record and its relationships
Summary → sets of related Records
Query   → datasets
```

## Result Type

Workbench infers a Formula's result type when practical. When the result type cannot be determined reliably, Workbench may allow or require the user to specify the intended result type.

## Recalculation

Formula values are live. Changes to referenced values or relationships trigger recalculation of dependent Formula values.

Formula dependencies participate in Workbench's dependency system. Circular dependencies are not permitted.

## Errors

Formula evaluation failures are preserved as identifiable Formula errors rather than silently converted to blank values. Persistent Formula errors are surfaced through the Problems system.

## Editing

Formula results cannot be directly overwritten. The user changes a Formula result by editing its source values or Formula definition. Where appropriate, a Formula Field may be explicitly converted to ordinary stored values using **Convert to values**.

---

# 17. Generated Fields

A **Generated** Field contains values owned and maintained by Workbench rather than manually entered by the user. Generated Fields are read-only under normal editing.

## V1 Generated Modes

Workbench v1 supports the following Generated modes:

- **Sequence**
- **UUID**
- **Created Time**
- **Modified Time**

## Sequence

Sequence generates an identifier when a Record is created.

A Sequence may use simple numeric values:

```text
1
2
3
```

or formatted values:

```text
INV-0001
INV-0002
INV-0003
```

Sequence configuration belongs to the Field. Generated sequence values are not based on the current visual row number. Reordering, filtering, sorting, or deleting Records does not redefine the identity of existing sequence values.

## UUID

UUID generates a unique identifier when a Record is created.

For example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Once generated, the UUID for a Record remains stable.

Sorting, filtering, moving, or otherwise changing the Record does not regenerate its UUID.

UUID values are distinct from Workbench's own internal Record identifiers. A UUID Field exposes a generated identifier as part of the user's Table schema, while Workbench's internal identity system remains implementation-managed.

## Created Time

Created Time records when a Record was created. Workbench owns and maintains this value.

## Modified Time

Modified Time records when a Record was last modified according to Workbench's Record modification semantics. Workbench owns and maintains this value.

The exact definition of which operations count as a Record modification may be refined during implementation.

## Editing

Generated values cannot ordinarily be manually overwritten. Where appropriate, a Generated Field may be explicitly converted to ordinary stored values using **Convert to values**. After conversion, Workbench no longer maintains the generated behavior.

## Future Modes

Additional Generated modes may be introduced later, such as:

- Created By
- Modified By

User/account-dependent modes such as Created By and Modified By are outside v1.

---

# 18. Attachments and Files

Files are supporting resources stored within a workbench. An **Attachment** Field creates relationships between Records and those File resources.

## File Resources

A File has its own stable identity within the workbench. In v1, File contents are embedded within the `.wkbn` artifact.

Files are stored at the workbench level and are presented through the top-level Files section rather than through Collections.

## Attachment Values

An Attachment value contains zero or more references to File resources. 

The Attachment does not duplicate the underlying File contents. For example, if the same File is attached to several Records, those Records may reference the same File resource rather than storing multiple copies of its bytes.

## File Support

Workbench v1 intentionally provides limited built-in File editing and viewing. Plain text files may be viewed and edited directly where supported. Other File types may be represented within Workbench and opened using an appropriate external application.

Workbench is not intended to become a general-purpose editor for documents, PDFs, images, video, or other arbitrary File formats.

## Embedded Storage

V1 Files are embedded in the workbench. This allows a `.wkbn` artifact to remain portable and self-contained.

The File model should allow future storage strategies, such as linked external Files, without requiring File resources to become a different conceptual object.

## File Dependencies

Resources that depend on a File reference its stable File identity. Deleting or trashing a File that is referenced elsewhere is dependency-aware. Workbench warns the user and identifies affected dependencies rather than silently breaking or cascading them.

## File Safety

Embedded Files are treated as data. Including a File in a workbench does not grant that File executable privileges within Workbench. External opening of Files follows the operating system's normal application and security behavior.

---

# 19. Required Fields and Validation

Workbench favors preserving user-entered data and clearly identifying problems over silently discarding values or unnecessarily blocking data entry.

## Required Fields

Normal stored Fields may be configured as **Required**.

A Required Field is expected to contain a valid non-blank value for each applicable Record. If a Record does not satisfy a Required constraint, Workbench preserves the Record and surfaces the violation through the Problems system.

Required does not imply that Workbench must reject creation or editing of an incomplete Record. This allows users to work incrementally while retaining visibility into incomplete data.

## Semantic Validation

Field types may define validation appropriate to their semantics.

Examples include:

- Email syntax
- URL syntax
- Numeric requirements
- Choice membership
- Reference validity
- JSON validity

When practical, invalid user input is preserved so that it can be inspected and corrected. Workbench should not silently replace an invalid value with a blank value or another unrelated value.

## Problem State

A value that violates persistent validation rules may enter a Problem state. The original user input remains visible or recoverable where practical. Correcting the underlying value or configuration automatically resolves the associated Problem when the constraint is satisfied.

## Validation and Data Entry

Validation should communicate data quality without making routine data entry unnecessarily rigid. Workbench may prevent states that cannot be represented safely, while recoverable semantic errors should generally remain visible and correctable.

---

# 20. Field Type Conversion and Defaults

Workbench supports changing Field types while prioritizing preservation of existing data.

## Field Type Conversion

When the user changes a Field's type, Workbench attempts to convert existing values to the new type. Values that can be converted successfully are converted. Values that cannot be converted are preserved in an identifiable Problem state rather than silently discarded.

For example, changing a Text Field containing:

```text
10
25
Unknown
42
```

to Number may produce valid numeric values for `10`, `25`, and `42` while preserving `Unknown` as a value requiring correction.

The user should be able to understand the expected outcome of a potentially destructive or broad conversion before committing it.

## Compatible Type Changes

Some Field type changes may require little or no underlying value conversion because the types share compatible semantics. Other changes represent true conversions and may produce invalid values. Workbench may distinguish these cases in the user experience.

## Convert to Values

Derived or system-owned Fields may support an explicit **Convert to values** operation. This operation snapshots the Field's current results into ordinary stored values and removes the original derivation or generation behavior.

This may apply to Field types such as:

- Formula
- Summary
- Generated
- other appropriate derived Fields

Convert to values is an explicit operation because it changes the meaning and future behavior of the Field.

## Default Values

Normal stored Fields may define an optional default value. A default is applied when a new Record is created. Changing a Field's default does not rewrite existing Records.

Default values do not eliminate the distinction between blank and explicit values such as:

```text
false
0
""
```

Derived and system-owned Fields do not use ordinary Field defaults when their values are controlled by their derivation or generation behavior.

---

# 21. Record Operations

Records are structured entities within a Table. Workbench supports creating, editing, duplicating, and deleting Records while preserving stable Record identity and dependency awareness.

## Creating Records

Records may be created through actions including:

- entering data into a new Grid row
- pasting tabular data
- explicit Record creation
- Import.

A Record receives a stable internal identity when it is created. Its identity does not depend on its current row number, position, sort order, or displayed values.

## Editing Records

Editing a Record changes its stored values.

Derived Fields such as Formula, Summary, and Generated Fields are not directly editable unless explicitly converted to ordinary stored values.

Changes to stored values propagate through dependent References, Formulas, Summaries, Queries, Views, and Dashboards as appropriate.

## Duplicating Records

Duplicating a Record creates a new Record with its own stable identity.

Workbench copies editable stored values from the source Record. References and Attachments are copied as relationships to the same target Records or Files.

Workbench does not automatically duplicate related Records or attached File contents. Derived and system-owned values are recalculated or regenerated as appropriate.

For example:

- Formula values recalculate
- Summary values recalculate
- Generated UUID values regenerate
- Generated Sequence values regenerate
- Created Time reflects creation of the new Record

## Deleting Records

Deleting a Record removes it from the active Table data. Deleted Records are recoverable through Table history. Record deletion does not use the resource-level Trash interface.

If other resources reference a deleted Record, Workbench preserves those relationships as identifiable broken dependencies rather than silently replacing them with unrelated blank values. Broken dependencies are surfaced through the Problems system.

## Bulk Operations

Workbench may apply Record operations to multiple selected Records. Bulk operations should behave transactionally where practical so that failures do not leave large edits in an unintentionally partial state.

---

# 22. Views

A **View** is a saved presentation of a single Table. A View does not own or duplicate the Table's underlying Records or Fields. Changes to Table data are therefore reflected across all Views of that Table.

## View Responsibilities

A View may control presentation behavior including:

- Field visibility
- Field order
- Field widths
- frozen Fields
- filters
- sorting
- grouping
- row height and density
- conditional formatting
- other presentation-specific settings

The general boundary is:

> If it changes what the data means, it belongs to the Table. If it changes which data is shown or how it is presented, it belongs to the View.

## Saved Configuration

View configuration is saved automatically. Filtering, sorting, grouping, and other View changes do not exist as a separate unsaved presentation state. A user who wants to preserve one presentation while experimenting with another may create or duplicate a View.

Normal undo behavior may also reverse View configuration changes where appropriate.

## View Types

View Type is an explicit property of the View model. Workbench v1 ships with **Grid View** as its primary View Type.

The View architecture should allow additional View Types to be introduced later without redefining the relationship between Tables and Views.

Potential future View Types include:

- Kanban
- Calendar
- Gallery
- other specialized presentations

## Grouping

Grid Views support grouping without introducing hierarchy into the underlying Records.

Grouping is presentation behavior. V1 supports multi-level grouping. Groups may display View-level aggregates such as:

- Count
- Sum
- Average
- Minimum
- Maximum

These presentation aggregates are distinct from Summary Fields and do not become stored Table Fields.

## Conditional Formatting

Conditional formatting belongs to a View. Different Views of the same Table may therefore present the same underlying values using different formatting rules.

## Default View

Every Table has a default View. When a Table is opened without a specific View, Workbench opens its last-used View when available and otherwise opens its default View.

---

# 23. Queries

A **Query** is a saved, live, read-only derived dataset constructed from one or more Tables and/or other Queries. Queries allow users to transform, combine, filter, calculate, group, and summarize structured data without modifying the underlying source data.

## Query and View Boundary

A View presents one Table. A Query constructs a derived dataset.

A View may filter, sort, group, or present a Table differently without changing the fundamental dataset being viewed.

A Query may perform operations such as:

- selecting or projecting Fields
- filtering Records
- joining datasets
- calculating derived values
- grouping
- aggregation
- sorting
- combining multiple sources

## Visual Query Builder

The primary Query authoring experience in v1 is a visual Query builder. Users should not need to write SQL to create Queries.

Query definitions are stored as structured Workbench definitions rather than opaque SQL strings. SQL may be considered as an advanced authoring option in a future version.

## Transformation Pipeline

A Query is conceptually an ordered transformation pipeline. For example:

```text
Source
  ↓
Filter
  ↓
Join
  ↓
Calculate
  ↓
Group
  ↓
Sort
  ↓
Result
```

Transformation order matters. The Query interface may allow the user to inspect or preview the intermediate dataset produced at a selected step.

## Sources

A Query may use tables or other queries as a source. Query dependencies form a directed dependency graph. Circular Query dependencies are not permitted.

If an upstream Query or Table becomes unavailable or invalid, dependent Queries preserve the broken dependency and surface an appropriate Problem.

## Relationships and Joins

Existing Reference relationships should be understood by the Query system and may be suggested when combining related data. Queries may also support appropriate ad-hoc joins between compatible Fields when no modeled Reference relationship exists. Detailed join semantics and advanced join behavior are implementation and later product-design decisions.

## Live Results

Queries are live. Changes to source data propagate to Query results and onward to dependent resources.

## Read-Only Results

Query results are read-only in v1. Where a result can be traced to an underlying source Record, Workbench should provide convenient navigation to that source for editing.

---

# 24. Dashboards

A **Dashboard** is a saved presentation composed of Widgets that visualize or summarize Workbench data. Dashboards consume live data from Tables and Queries. They do not own or modify their source data.

## Widgets

A Dashboard contains one or more Widgets. A Widget defines a particular presentation of data from an appropriate Workbench source.

V1 should support core Widget categories such as:

- charts
- KPI or value cards
- tabular data
- text and labels

The exact set of chart types and Widget configurations may be refined during Dashboard implementation.

## Data Sources

Widgets may use Tables or Queries as data sources where appropriate. Queries are the preferred mechanism when a Widget requires a dataset substantially different from the underlying Table.

## Live Data

Dashboard Widgets reflect changes to their source data. Changes to Tables, Formulas, Summaries, References, or Queries propagate to dependent Dashboard Widgets as appropriate.

## Presentation Boundary

Dashboards are presentation resources. Dashboard configuration may control:

- Widget placement
- Widget size
- visualization configuration
- labels and titles
- presentation formatting

Dashboard configuration does not redefine the underlying source data.

## Broken Sources

If a Widget's source becomes unavailable or invalid, the Widget preserves the dependency where practical and displays an appropriate broken state. Persistent broken Dashboard dependencies are surfaced through the Problems system.

## V1 Scope

V1 focuses on composing and presenting analytical information. Dashboard-driven editing of source Records is not part of the v1 Dashboard model. Advanced interactive Dashboard behavior, global Dashboard filtering, and more specialized visualization capabilities may be introduced later.

---

# 25. Problems

The **Problems** system provides a workbench-wide view of persistent, actionable issues affecting data or resource correctness. Problems are not general notifications. They represent conditions that remain relevant until the underlying issue is resolved or no longer exists.

## Examples

Problems may include:

- invalid Field values
- Required Field violations
- failed Field type conversions
- broken References
- Formula errors
- invalid Field configuration
- broken Query dependencies
- invalid Query definitions
- broken Dashboard Widget sources
- missing or broken File dependencies

Problems should not be created for ordinary transient events such as:

- import completed
- workbench saved
- Records copied
- export completed
- routine informational messages

Those events should use appropriate transient feedback instead.

## Severity

V1 supports two Problem severity levels:

- **Error**
- **Warning**

Additional severity categories should only be introduced if a meaningful product need emerges.

## Status Bar

The Status Bar exposes the current Problem state of the workbench.

For example:

```text
⚠ 3 Problems
```

Selecting the Problems indicator opens the Problems interface in the Bottom Panel.

## Problems Interface

The Problems interface provides a workbench-wide list of active Problems. Each Problem should identify enough context to understand where the issue exists, such as:

- Table
- Field
- Record
- Query
- Dashboard
- Widget
- File
- other affected resource

Problems may be filtered or grouped where useful.

## Navigation

Selecting a Problem navigates to the affected resource and location when possible. If the appropriate target is already open, Workbench activates it and navigates to the relevant location. If it is not open, Workbench may open the appropriate resource in preview and navigate to the affected location.

## Resolution

Problems persist while their underlying conditions remain unresolved. When the underlying condition is corrected, Workbench automatically removes the corresponding Problem. Users should not need to manually dismiss a Problem that still represents an actual data or model issue.

## Architecture

Problems should be derived from authoritative Workbench state rather than functioning as an unrelated collection of warning messages. The Problems system should remain capable of handling large workbenches without requiring the entire dataset to be loaded into the frontend.

---

# 26. Import

**Import** interprets external structured data and creates Workbench-native structured data from it. Import is distinct from uploading a File.

- **Import** creates structured Workbench data.
- **Upload** embeds an external File as a File resource.

The same external file may support either action depending on the user's intent. Workbench does not determine the action solely from the file extension.

## V1 Import Scope

In v1, Import creates a new Table. Import does not merge into, update, or synchronize with an existing Table.

Every imported Table requires a Table name. Workbench may suggest a name derived from the source filename, but the user can confirm or change it before creation.

## Import Configuration

Before committing an Import, Workbench provides a preview and configuration experience.

Where applicable, this includes:

- detected columns
- sample Records
- inferred Field names
- inferred Field types
- header configuration
- delimiter configuration
- text encoding
- other format-specific parsing options

The user may adjust the inferred structure before creating the Table.

## Type Inference

Workbench may infer Field types from imported data. Inference is assistance rather than an irreversible schema decision. The user should be able to review and change inferred Field types before the Table is created.

## Transactional Import

Import is transactional. Cancelling or failing an Import must not leave behind an unintentionally partial Table. The operation either creates the intended Table successfully or leaves the workbench in a known-good state.

## Large Imports

Large Imports execute away from the UI execution path. Progress should be visible, the application should remain responsive, and the operation should be cancellable where practical. Import architecture must not require the entire imported dataset to be held in frontend memory.

---

# 27. Export

**Export** creates an external representation of Workbench data for interoperability with other applications and systems. Export is distinct from native Workbench sharing.

- **Export** targets external formats and interoperability.
- **Native sharing** preserves Workbench-specific structure and semantics.

## Resource-Scoped Export

Export begins from the resource the user intends to export. Supported export behavior depends on the resource type and selected external format.

## Exporting a View

When exporting a View, the default export represents the data visible through that View. This may include the View's:

- filtered Records
- visible Fields
- Field order
- applicable sorting
- other relevant presentation choices

The export experience provides an **Include all data** option. When enabled, Workbench exports the complete underlying Table dataset rather than limiting the export to the current View presentation.

## Exporting Other Resources

Queries export their resulting datasets where supported. Other resource types may support appropriate external export formats when those formats provide meaningful interoperability.

## Derived Values

When exporting tabular data, derived values such as Formula and Summary results may be exported as their current resulting values where appropriate. The external format does not need to preserve Workbench-specific derivation semantics unless explicitly supported.

## Large Exports

Large Exports execute away from the UI execution path. Export should provide progress where useful, remain cancellable where practical, and avoid requiring the complete dataset to be loaded into frontend memory.

---

# 28. Native Sharing

Workbench v1 supports native sharing of Workbench resources. Native sharing preserves Workbench-specific structure and semantics that may be lost through ordinary external export.

Native sharing uses a portable Workbench package format separate from the `.wkbn` workbench artifact itself. The exact package name and file extension are not defined by this specification.

## Package Contents

A native package may contain one or more explicitly selected resources.

Workbench includes dependencies required for those resources to function correctly. Dependencies are resolved recursively where necessary. For example, sharing a Dashboard may require inclusion of:

```text
Dashboard
    ↓
Queries
    ↓
Tables
    ↓
Fields / required relationships
```

The package should include only the resources and dependencies required for the selected sharing operation rather than automatically including the entire workbench.

## Importing a Native Package

Importing a native package creates local Workbench resources. V1 uses safe duplication rather than attempting name-based merging with existing resources. Imported resources receive new local stable identities. References and other internal dependencies within the package are remapped to the newly created local identities.

## Naming Conflicts

If imported resources conflict with existing names, Workbench resolves those conflicts without overwriting unrelated existing resources. The user should be able to understand what resources will be added before committing the import.

## Snapshot Semantics

Native packages are snapshots. Importing a package does not establish a live connection to the workbench from which it originated. Subsequent changes in either workbench do not automatically propagate to the other.

## Inspectability

Before importing a native package, Workbench provides enough information for the user to understand what the package contains. This may include:

- included resource types and counts
- resource names
- embedded File names, types, and sizes
- total package size
- package format version
- integrity or validation status

The same inspectability principle applies when preparing native packages for sharing.

## Future Sharing

Live sharing, synchronization, collaboration, users, and permissions are outside v1. The v1 local data model should not introduce unnecessary account or cloud identity concepts solely in anticipation of those future features.

---

# 29. Trash and Deletion

Workbench distinguishes between resource-level deletion and Record-level deletion.

## Resource Trash

Deleting a resource places it in Workbench's Trash where appropriate. Trash provides a recovery path for accidentally deleted resources.

Resources managed through Trash include appropriate top-level or organizational resources such as:

- Tables
- Queries
- Dashboards
- Files
- Collections

Views and other owned resources should follow deletion behavior appropriate to their owning resource and recovery model.

## Dependency-Aware Deletion

Deletion is dependency-aware. Before trashing a resource that is referenced by other resources, Workbench warns the user and identifies affected dependencies where practical.

Workbench does not silently cascade deletion through dependent resources. For example, trashing a Table does not automatically delete every Query or Dashboard that depends on it. Instead, affected dependencies become identifiable broken dependencies and are surfaced through the Problems system where appropriate.

## Restore

Resources in Trash may be restored. Restoration should preserve the resource's stable identity where practical so that existing dependencies can become valid again.

If the resource's previous organizational location is no longer available, Workbench restores it to an appropriate valid location and informs the user where necessary.

## Permanent Deletion

Trash supports permanent deletion. Permanent deletion is an explicit destructive action. Workbench should clearly communicate when permanent deletion may make recovery impossible through ordinary Trash behavior.

History or recovery mechanisms may still retain implementation-level recovery information according to their own retention rules.

## Record Deletion

Records do not use the resource-level Trash interface. Deleted Records are recoverable through Table history.

References to deleted Records remain identifiable as broken relationships rather than silently becoming unrelated blank values.

---

# 30. History and Recovery

Workbench maintains durable history to support recovery from mistakes and destructive changes. History is distinct from ordinary undo and redo.

## Undo and Redo

Undo and redo are scoped to the active resource rather than implemented as one global chronological sequence for the entire workbench. This prevents unrelated edits across multiple open resources from becoming unnecessarily coupled through a single undo stack. The exact grouping of edits into undoable operations may depend on the resource and operation type.

## Durable History

Workbench maintains durable history beyond the immediate undo/redo stack.

History supports recovery scenarios such as:

- restoring deleted Records
- recovering from an accidental bulk edit
- inspecting meaningful prior changes
- recovering an earlier state of appropriate resources

History survives ordinary application restarts.

## Recovery

History is a user-facing v1 recovery feature. Users should be able to recover from meaningful mistakes without needing to understand Workbench's internal persistence system. Recovery operations should preserve data integrity and dependency relationships where practical.

## History and Autosave

Autosave and History work together. Persisting a change automatically does not mean the previous state immediately becomes unrecoverable. Workbench should create meaningful recovery boundaries without requiring the user to manually save versions.

## History Is Not Version Control

V1 History is not intended to provide a Git-style version-control system.

V1 does not require:

- branches
- commits
- merge workflows
- named versions
- source-control-style diffs

Those concepts may be considered separately if future product needs justify them.

## Retention

The exact retention strategy, storage limits, compaction behavior, and recovery granularity are implementation decisions that must balance:

- useful recovery
- `.wkbn` artifact size
- performance
- long-term reliability

The implementation must not allow history growth to make normal workbench usage impractical.

---

# 31. `.wkbn` Artifact

A workbench is represented to the user as a single portable `.wkbn` artifact. For example:

```text
Finance.wkbn
```

The artifact is application-managed and contains the structured data, embedded Files, metadata, and other state required for the workbench.

## User Model

From the user's perspective, a `.wkbn` artifact behaves as one self-contained workbench. Users should be able to:

- move it
- rename it
- copy it
- back it up
- transfer it to another computer
- open it with Workbench

Ordinary use should not require the user to manage Workbench's internal storage structure.

## Internal Structure

Internally, `.wkbn` is a structured package or container rather than one undifferentiated data blob. Conceptually:

```text
Finance.wkbn
├── database
├── files/
├── metadata
└── package / integrity information
```

The exact internal package format is an implementation decision.

## Structured Data

SQLite is the primary structured-data persistence technology within the `.wkbn` artifact. Structured Workbench state should be stored in a form appropriate for transactional access, querying, indexing, migration, and recovery.

## Embedded Files

Embedded File contents are stored separately from the primary structured database within the `.wkbn` container. Large arbitrary File contents should not be stored as giant database blobs unless a specific technical reason justifies doing so. The database stores the metadata and identities required to associate embedded File contents with Workbench File resources.

## Portability

A valid `.wkbn` artifact should contain everything required for ordinary offline use of that workbench. Moving the artifact should not silently leave required embedded data behind elsewhere on the machine.

Temporary runtime data, caches, or recoverable implementation state may exist outside the artifact when necessary, but the `.wkbn` remains the authoritative portable workbench representation.

---

# 32. Inspectability and Safety

Workbench artifacts and native sharing packages are application-managed but transparently inspectable through Workbench. Users should be able to understand what an artifact or package contains without directly editing its internal representation.

## Contents Inspection

Workbench provides a contents inspection experience where appropriate. Information may include:

- resource counts
- resource names and types
- embedded File names
- embedded File types
- embedded File sizes
- total artifact or package size
- format or schema version
- integrity or validation status

The amount of detail presented may depend on whether the user is inspecting an entire `.wkbn` artifact or a native sharing package.

## Sharing Safety

Before sharing an entire workbench or native package, users should be able to inspect the contents that will be included. This is particularly important for embedded Files that may contain information not immediately visible from the currently active Table, View, Query, or Dashboard.

## Import Safety

Before importing a native Workbench package, Workbench should allow the user to understand what resources and Files it contains. Workbench validates the package before committing imported resources. Invalid, corrupted, or unsupported packages must not be allowed to partially modify the destination workbench.

## Embedded File Safety

Embedded Files are treated as data and do not gain executable privileges merely by being contained within a Workbench artifact. Workbench should not automatically execute embedded scripts, binaries, macros, or other active content merely because an artifact has been opened.

## Internal Editing

Direct manual editing of `.wkbn` internals is not a supported user workflow. Inspectability means that Workbench makes contents understandable; it does not require the internal package format itself to function as a user-editable directory structure.

---

# 33. Offline-First

Workbench is offline-first. Core Workbench functionality must remain available without an internet connection.

## Offline Core

Without network access, users must be able to perform ordinary core operations including:

- create and open workbenches
- create and edit Tables and Records
- create and configure Views
- use Formulas and Summaries
- use References
- create and execute Queries
- use Dashboards
- import and export data
- use native sharing packages
- work with embedded Files
- use History and recovery
- inspect and resolve Problems

Core structured-data workflows must not depend on a remote service being available.

## Local Authority

The local `.wkbn` artifact is authoritative for v1 workbench state. Workbench must not require a cloud account, remote database, license server connection, or synchronization service in order to access ordinary local workbench data.

Commercial licensing requirements of the underlying desktop framework, if any, are an application distribution concern and must not turn individual workbench data into a cloud-dependent resource.

## Network Features

Features that inherently require network access may use it. Future examples may include:

- collaboration
- synchronization
- cloud backup
- remote data sources
- external integrations

Failure or unavailability of those services should not prevent unrelated local Workbench functionality from operating.

## Future Synchronization

Although synchronization is outside v1, Workbench should avoid architectural decisions that unnecessarily prevent future sync. Stable identities, explicit dependencies, durable history, and clear persistence boundaries should provide a foundation for future synchronization without making cloud concepts part of the v1 product model.

---

# 34. Persistence and Crash Safety

Workbench prioritizes preservation of user data and recovery from interrupted operations. A crash, power loss, forced termination, or interrupted write should not make an otherwise healthy workbench unusable.

## Transactional Persistence

Operations that modify persistent state should use transactional behavior where appropriate. This is especially important for operations such as:

- Import
- schema changes
- Field type conversions
- File uploads
- native package imports
- migrations
- large bulk edits
- history updates

An interrupted operation should either commit successfully or leave the workbench in a known-good recoverable state.

## Autosave

Autosave writes changes without requiring explicit user action. Autosave must not trade recoverability for convenience.

Persistence should use durable mechanisms appropriate to the underlying storage technology rather than relying only on in-memory state or delayed frontend serialization.

## Abnormal Shutdown

Workbench should detect evidence of an abnormal previous shutdown where practical. When necessary, Workbench automatically verifies the workbench and performs safe recovery before allowing ordinary modification. The user should not need to understand database implementation details to recover from an ordinary application crash.

## Corruption

If Workbench detects corruption it cannot safely repair automatically, it should avoid making further destructive modifications. Where possible, Workbench should:

- preserve the affected artifact
- explain that recovery is required
- provide or guide the user toward available recovery options

Workbench must prefer preserving recoverable data over attempting speculative destructive repair.

## Atomic Package Changes

Changes involving both structured database state and embedded File contents must be coordinated so that the `.wkbn` artifact does not intentionally enter a persistent half-updated state. The exact transaction and packaging mechanism is an implementation decision.

---

# 35. Format Versioning and Migration

Every `.wkbn` artifact has an explicit Workbench format and/or schema version. Workbench uses version information to determine whether an artifact can be opened directly, requires migration, or is unsupported by the running application version.

## Migrations

When Workbench's persisted data model changes, supported older artifacts are upgraded through deterministic migrations. Migrations must be:

- explicit
- ordered
- testable
- transactional where practical
- safe to retry or recover from appropriately

Migration logic should not depend on manually interpreting individual user artifacts.

## Recovery Before Migration

Before performing a migration that modifies an existing `.wkbn` artifact, Workbench creates or preserves an appropriate recovery point. If migration fails, the original usable state should remain recoverable. A failed migration must not intentionally leave the only copy of the workbench in a partially migrated state.

## Newer Unsupported Formats

An older Workbench application may encounter a `.wkbn` artifact created by a newer version whose format it does not understand. In that case, the older application must not attempt to modify the artifact as though it understood the newer format. Workbench should explain that a newer compatible application version is required.

Read-only or compatibility behavior may be introduced when technically safe, but unsupported modification is prohibited.

## Compatibility

Format evolution should preserve user data and semantics wherever possible. Removing or changing a feature must not silently reinterpret existing persisted data as something materially different.

## Native Packages

Native sharing packages also carry sufficient format/version information for Workbench to determine whether they can be safely inspected and imported. Package compatibility and `.wkbn` artifact compatibility may evolve independently where appropriate.

---

# 36. Scale Requirements

Workbench is designed for datasets substantially larger than those typically comfortable in conventional spreadsheet applications. The architecture must not impose an artificial Record ceiling simply because the frontend cannot hold or render the entire Table.

## V1 Scale Targets

Workbench uses the following engineering targets:

| Record Count | Expectation                                                   |
| -----------: | ------------------------------------------------------------- |
|    1 million | Routine workload                                              |
|   10 million | Primary v1 engineering target                                 |
|   50 million | Stress target                                                 |
| 100 million+ | Best effort; architecture should not fundamentally prevent it |

### 1 Million Records

Tables containing approximately one million Records should represent ordinary supported workloads. Common navigation and interaction should remain practical.

### 10 Million Records

Ten million Records is the primary large-scale v1 benchmark. Workbench should be intentionally designed and tested around this scale. The application should remain usable without requiring the entire Table to be loaded into frontend memory.

### 50 Million Records

Fifty million Records is a stress target. Workbench should be capable of opening and navigating appropriately structured workbenches at this scale. Expensive operations may take noticeable time. Responsiveness expectations may differ from smaller datasets, but dataset size alone should not make the application architecture fail.

### 100 Million+ Records

Datasets of 100 million Records or more are best effort in v1. Workbench does not guarantee that every operation will remain interactive at this scale. However, the architecture should avoid arbitrary assumptions that fundamentally prevent datasets from growing beyond the primary benchmark.

## Performance Principle

Record count must not translate directly into frontend memory consumption or rendered UI elements. Large Tables remain primarily within the data layer.

The frontend operates on bounded windows, pages, summaries, and other appropriately sized results. Operations such as sorting, filtering, searching, and grouping large datasets should execute in the data/query layer rather than by materializing entire Tables as frontend arrays.

---

# 37. Bounded Frontend Data

The Workbench frontend must operate on bounded amounts of data. UI components do not own complete large Table datasets.

## Data Access

The frontend requests the data required for the current interaction from the Workbench data/domain layer. Examples include:

- visible Grid Records
- additional Records needed during scrolling
- Query result windows
- group summaries
- search results
- Dashboard result sets
- metadata required for the current resource

The data layer returns appropriately bounded results.

## Grid Data

The Grid renders only the Records and Fields required for the visible region and appropriate surrounding buffers. Scrolling through a large Table must not require constructing a frontend representation of every Record.

## Domain Boundary

Frontend components do not freely access SQLite or internal `.wkbn` storage. Persistent data access occurs through defined Workbench data/domain interfaces.

Conceptually:

```text
React UI
    ↓
Workbench domain / data API
    ↓
Data and query engine
    ↓
SQLite / .wkbn persistence
```

This boundary allows persistence, caching, query execution, migrations, history, and other data behavior to evolve without coupling UI components directly to storage implementation details.

## Derived Data

The same bounded-data principle applies to Queries, Dashboards, Problems, Search, and other features that may operate across large datasets. The frontend should receive the result required for presentation rather than the entire source dataset used to calculate that result.

---

# 38. Long-Running Operations

Expensive Workbench operations must not block the user interface execution path. Examples may include:

- large Imports
- large Exports
- complex Queries
- expensive recalculation
- Field type conversion
- large bulk edits
- native package creation or import
- integrity verification
- migrations

## Responsiveness

The Workbench UI should remain responsive while long-running operations execute. The implementation may use separate processes, workers, threads, or other appropriate execution mechanisms. The exact mechanism is an implementation decision.

## Progress

Operations that take meaningful time should expose progress or activity state where practical. Progress may be surfaced through appropriate application UI such as the Status Bar, Bottom Panel, notifications, or operation-specific interfaces.

## Cancellation

Long-running operations should be cancellable when safe and practical. Cancellation must not intentionally leave persistent Workbench state partially modified or corrupted. Where an operation cannot safely be cancelled at a particular point, Workbench may complete the required atomic portion before cancellation takes effect.

## Failure

Failure of a long-running operation should leave the workbench in a known-good state. Errors should provide enough information for the user to understand what failed and whether any action is required.

---

# 39. Desktop Technology

Workbench is a cross-platform desktop application.

## Provisional Desktop Runtime

The preferred v1 desktop runtime is **MōBrowser 2.x**.

MōBrowser provides a TypeScript/Node.js desktop architecture with Chromium-based rendering and native application capabilities. This selection is provisional until technical validation confirms that it satisfies Workbench's requirements.

## Validation Criteria

Before becoming an irreversible architectural dependency, the desktop runtime should be validated against requirements including:

- Windows support
- macOS support
- Linux support
- React integration
- SQLite integration
- native File access
- `.wkbn` artifact handling
- multiple application windows
- process isolation
- long-running background work
- native module support
- packaging and distribution
- development and debugging workflows
- licensing and commercial distribution requirements
- performance at Workbench's intended scale

## Framework Isolation

Core Workbench domain logic should not depend unnecessarily on MōBrowser-specific APIs. Desktop-runtime-specific functionality should be isolated behind appropriate application boundaries.

Conceptually:

```text
Workbench Domain
        ↓
Application Services
        ↓
Desktop Runtime Integration
        ↓
MōBrowser
```

This reduces the cost of changing desktop runtimes if future requirements demand it.

## Alternatives

If MōBrowser does not satisfy Workbench's technical, licensing, stability, or distribution requirements, **Electron** is the primary fallback. The Workbench domain model and persistence architecture should not require redesign merely because the desktop shell changes.

---

# 40. Frontend Technology

Workbench uses **React with TypeScript** for its primary application UI. React provides the component model and UI ecosystem for the Workbench renderer.

## Architectural Boundary

React is the UI framework. It is not the Workbench domain model, persistence layer, query engine, or data architecture. Domain behavior should not be encoded solely as React component state.

Conceptually:

```text
React
    ↓
UI / interaction layer
    ↓
Workbench application and domain APIs
    ↓
Data / query / persistence systems
```

## State Management

The exact supporting state-management libraries are implementation decisions. Workbench should choose them according to actual application needs rather than making a state-management library part of the product architecture prematurely.

## Build Tooling

Vite or an equivalent modern TypeScript build system may be used for renderer development. The exact build tooling is an implementation choice rather than a product requirement.

## Performance-Sensitive UI

React does not require every performance-sensitive surface to render every element through ordinary React component reconciliation. Specialized components such as the Grid may use optimized rendering techniques where necessary while remaining integrated with the React application shell.

---

# 41. Grid Architecture

The Grid is a specialized high-performance Workbench surface. It must provide spreadsheet-like interaction while operating over structured, potentially very large datasets.

## Virtualization

The Grid renders only the visible region and an appropriate surrounding buffer. Neither Record count nor Field count should translate directly into an equivalent number of mounted frontend components.

## Data Access

The Grid requests bounded data from the Workbench data layer. Scrolling causes additional data windows to be requested as necessary. The Grid must not require the entire Table to be materialized in frontend memory.

## Interaction

The Grid should support familiar high-performance interactions including:

- cell selection
- range selection
- keyboard navigation
- direct editing
- copy
- paste
- fill or appropriate bulk operations
- Record selection
- Field interaction

## Structured Semantics

Grid coordinates are presentation coordinates. The underlying semantic identity remains based on Records and Fields. For example, selecting the visible third row and fourth column does not make `D3` a persistent data identity.

## Large Operations

Operations initiated through the Grid may affect datasets larger than the currently rendered selection. Large operations should be expressed to the domain/data layer rather than implemented by iterating through frontend-rendered cells.

## Rendering Strategy

The exact Grid rendering technology is an implementation decision. Workbench may use:

- optimized DOM rendering
- canvas rendering
- hybrid rendering
- another appropriate approach

The choice should be validated against interaction quality, accessibility, maintainability, and Workbench's scale targets.

---

# 42. Architectural Separation

Workbench separates product semantics from presentation, desktop runtime, and persistence implementation. The architecture should maintain clear responsibilities between major layers.

Conceptually:

```text
Desktop Shell
      ↓
React UI
      ↓
Application Services
      ↓
Workbench Domain
      ↓
Data / Query Engine
      ↓
Persistence
      ↓
.wkbn Artifact
```

This diagram represents responsibility boundaries rather than requiring a particular process layout.

## Domain Layer

The domain layer represents Workbench concepts such as:

- Tables
- Fields
- Records
- Views
- References
- Formulas
- Summaries
- Queries
- Dashboards
- Files
- dependencies
- history

Domain behavior should not depend on how a particular UI component happens to display the resource.

## Application Services

Application services coordinate operations that cross domain or infrastructure boundaries.

Examples may include:

- Import
- Export
- native sharing
- migrations
- history and recovery
- File operations
- background operations
- workbench opening and closing

## Data and Query Layer

The data/query layer provides efficient access to structured data. Responsibilities include appropriate forms of:

- Record retrieval
- filtering
- sorting
- grouping
- Query execution
- aggregation
- indexing
- bounded result delivery

## Persistence Layer

The persistence layer manages durable `.wkbn` state. It coordinates SQLite, embedded File storage, package metadata, migrations, integrity, and transactional behavior.

## UI Layer

The UI layer presents Workbench state and translates user interaction into application/domain operations. The UI should not become the authoritative owner of persistent Workbench data.

## Runtime Integration

Desktop-runtime-specific APIs should remain isolated from domain semantics where practical. This keeps MōBrowser, Electron, or another future shell from becoming inseparable from the Workbench data model.

---

# 43. Templates

The Workbench architecture should remain compatible with templates, but a full template system is not required for v1.

## V1 Creation

The primary v1 workbench creation path is:

```text
New Workbench
    ↓
Empty Workbench
```

Workbench may ship with a small number of built-in examples or starter workbenches if they materially improve onboarding.

## Deferred Template System

V1 does not require:

- user-created template management
- template marketplaces
- cloud template synchronization
- template publishing
- template permissions
- template versioning infrastructure

## Future Foundation

Native Workbench sharing provides much of the foundation required for future templates. A future template may effectively represent a reusable packaged set of Workbench resources with appropriate creation semantics layered on top. Template-specific concepts should therefore not be added to the core v1 model unless required by an actual v1 use case.

---

# 44. Explicitly Deferred Features

The following features are intentionally outside the core Workbench v1 scope. Their exclusion does not imply that they will never be supported.

## Collaboration and Cloud

Deferred features include:

- real-time collaboration
- cloud synchronization
- user accounts as a Workbench data requirement
- permissions and sharing roles
- cloud-hosted workbenches
- live cross-device synchronization

## Cross-Workbench Relationships

V1 does not support live References, Formulas, Queries, or other dependencies across workbench boundaries. Cross-workbench data movement occurs through supported import, export, and native sharing mechanisms.

## Additional View Types

V1 ships with Grid View. Potential future View Types include:

- Kanban
- Calendar
- Gallery
- Timeline
- other specialized presentations

The View model should support future expansion without requiring these experiences in v1.

## Advanced Dashboard Capabilities

Deferred Dashboard features may include:

- complex global Dashboard filters
- Dashboard-driven Record editing
- application-builder behavior
- highly specialized visualization systems

## Advanced Query Authoring

SQL authoring is not required in v1. The visual Query builder is the primary v1 Query interface. Advanced SQL or other textual query modes may be introduced later.

## Linked External Files

V1 File resources contain embedded content. Linked or externally referenced File storage may be introduced later.

## Polymorphic References

V1 Reference Fields target one Table. References capable of targeting Records from multiple Tables are deferred.

## Geographic Field Type

A dedicated geographic or location Field type is not included in v1. Addresses and similar values may use Text Fields until Workbench provides meaningful geographic semantics.

## Full Template System

User-managed and distributed template infrastructure is deferred.

## General-Purpose Document Editing

Workbench is not intended to become a general-purpose:

- word processor
- PDF editor
- image editor
- video editor
- programming IDE
- notebook environment

Supporting Files may still be stored, inspected, referenced, or opened externally.

---

# 45. Remaining Meaningful Open Questions

The core Workbench v1 product model is sufficiently defined to begin implementation planning. Remaining decisions should be made when they become relevant to concrete UX or engineering work rather than expanding the specification indefinitely.

Meaningful unresolved areas include the following.

## Native Package Format

The exact format, extension, and internal representation of native resource-sharing packages remain undecided. The product semantics are already defined:

- snapshot-based
- dependency-aware
- inspectable
- portable
- safe duplication on import

## `.wkbn` Container Mechanics

The exact physical container representation of `.wkbn` remains an implementation decision. The chosen design must satisfy the established requirements for:

- portability
- transactional safety
- SQLite access
- embedded Files
- migration
- inspectability
- recovery
- performance

## Grid Rendering Technology

The exact Grid implementation remains open. DOM, canvas, hybrid, or another approach should be selected through prototyping and measurement against actual Workbench requirements.

## Query Execution Details

The product behavior of Queries is defined, but detailed execution mechanics remain open. This includes areas such as:

- join execution
- indexing strategy
- Query planning
- caching
- incremental recalculation

These should be determined through engineering design and performance testing.

## History Storage

The user-facing History and recovery requirements are defined. The exact persistence representation, retention policy, compaction strategy, and storage limits remain engineering decisions.

## Desktop Runtime Validation

MōBrowser is the preferred v1 desktop runtime but remains subject to technical validation. The decision should be revisited only if prototyping identifies a material issue involving capability, stability, licensing, distribution, development workflow, or performance.

---

# 46. V1 Success Standard

Workbench v1 succeeds if it delivers a coherent local-first structured-data environment rather than merely accumulating spreadsheet and database features.

A successful v1 should allow a user to:

1. create and manage portable `.wkbn` workbenches;
2. create structured Tables and work with them through a fast spreadsheet-like Grid;
3. use a broad semantic Field type system;
4. create relationships between Records;
5. derive values using Formulas and Summaries;
6. organize Tables, Queries, and Dashboards using Collections;
7. create multiple saved Views of Table data;
8. construct live derived datasets through a visual Query builder;
9. build Dashboards from live Workbench data;
10. import and export structured data;
11. embed and reference Files;
12. share Workbench-native resources through portable packages;
13. identify and navigate persistent data/model Problems;
14. recover from mistakes through Trash and History;
15. work fully offline for core workflows;
16. trust autosave and crash-safe persistence;
17. work with datasets far larger than a conventional frontend can hold in memory;
18. move or back up a workbench as a self-contained artifact.

The product should maintain the principle:

> **Feel like a spreadsheet; behave like structured data underneath.**

That principle should remain visible across the Grid, Field system, relationships, Queries, persistence architecture, and user experience. Workbench v1 should establish a foundation capable of growing into richer Views, synchronization, collaboration, integrations, templates, and other future capabilities without requiring the core structured-data model to be replaced.