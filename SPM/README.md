# Advance360 — SPM Module Redesign

## Strategic Performance Management: Graph-Based Architecture

**Version:** 3.2
**Date:** 2026-04-07
**Status:** Design Finalized — Pending Implementation Approval

---

## CHANGE ADDENDUM (V3.1 — 2026-04-04)

This addendum formalizes business updates requested after Version 3.0.

### A0) Superseding Update (2026-04-07)

- This subsection supersedes and extends the prior V3.1 addendum.

### A0.1) Element Type Defaults Become Strategy-Start Defaults

- Element type flags are now treated as **default starting values** when that type is added to a strategy.
- This applies to strategy-facing capabilities including:
  - `budgetable`
  - `trackable`
  - `has_kpis`
  - `has_risks`
  - `linkable_with_project`
  - `vision_aligned`
  - `quarterly_measurable`
- When configuring a strategy, the selected type inherits these defaults first.
- Strategy owners/editors may then override the values for that strategy without changing the master organization-level type.
- Therefore:
  - organization-level type = reusable default profile
  - strategy-level type configuration = effective behavior inside one strategy

### A0.2) Strategy Templates

- Organizations can define **strategy templates** for frequently used strategy designs.
- A strategy template stores:
  - selected element types
  - per-strategy effective capability defaults for each selected type
  - root eligibility
  - allowed parent type combinations
  - edge weight methods by parent/child type pair
  - strict hierarchy setting
  - discrepancy threshold
  - template metadata (name, description, category, owner, version)
- Creating a strategy now supports two entry modes:
  - **From template**
  - **From scratch**
- Choosing a template pre-populates the new strategy configuration and locks template-defined type settings during creation.

### A0.3) Project Linkability

- Element types now include `linkable_with_project`.
- Like Trackable/Budgetable/KPIs/Risks, this is a default value copied into strategy configuration and can be overridden there.
- If enabled in the strategy context, elements of that type may be linked to project records.
- If disabled in the strategy context, project linkage controls are hidden and linking is blocked for that type in that strategy.

### A0.4) Multi-Parent Within the Same Strategy

- The same SPM element may now be linked to **multiple parents within the same strategy**.
- An element still appears only once as a node within a strategy, but may have multiple incoming edges.
- Root means **zero incoming edges** within that strategy.
- Validation must prevent cycles, not multiple parents.
- Calculation, navigation, and permissions must all treat the graph as a true DAG rather than a tree.

### A0.5) Imported Elements and Sharing Model

- An element owner may explicitly share an element for reuse by other strategy owners/editors.
- A shared element that gets added into another strategy is treated as an **imported element in that strategy context**.
- Importing an element into a strategy does **not** transfer ownership of the element.
- Imported elements preserve source ownership for global fields, while the importing strategy team gains strategy-scoped permissions only.
- Permissions must distinguish:
  - owned elements
  - shared/importable elements
  - imported elements in a consuming strategy
  - exported data artifacts

### A1) Terminology: Indicators → KPIs

- In product/UI language, **Indicator** is renamed to **KPI**.
- Technical compatibility is preserved for existing APIs/tables that still use `indicator` naming.
- Documentation and mockups should present KPI as the default business term.

### A2) New Edge Weight Methods

Existing methods:

- `IMPACT` (1–10)
- `PERCENT` (0–100)
- `EQUAL` (auto)

New methods:

- `BUSINESS_VALUE` (auto): edge contribution derives from child element `business_value`.
- `BUDGETABLE` (auto): edge contribution derives from child element `planned_budget`.

Rules:

- For `BUSINESS_VALUE` and `BUDGETABLE`, manual edge weight input is hidden/disabled.
- Normalization follows existing sibling normalization behavior.
- If source value is missing or zero, contribution is treated as zero.

### A3) Build Configuration Simplification

- Strategy build configuration now defines allowed type participation, parent combinations, and effective capabilities per strategy.
- Multiple types may exist at the same effective depth as long as they follow configured parent rules.
- Validation focuses on root eligibility, allowed parent combinations, cycle prevention, and configured maximum depth.
- This applies to configuration and runtime build actions.

### A4) New Type Capability: Quarterly Measurable

New capability on SPM element type:

- `quarterly_measurable` (boolean)

When enabled for a type:

- Element detail exposes a **Quarterly** tab.
- Quarterly model supports:
  - `baseline_value`
  - yearly entries:
    - `year`
    - `Q1 target`, `Q1 actual`
    - `Q2 target`, `Q2 actual`
    - `Q3 target`, `Q3 actual`
    - `Q4 target`, `Q4 actual`

### A5) Quarterly Alert Threshold

- Existing strategy discrepancy threshold is reused for quarterly alerts.
- Direction-aware evaluation is required:
  - `HIGHER_BETTER`: alert if actual is below target beyond threshold.
  - `LOWER_BETTER`: alert if actual is above target beyond threshold.

### A6) Predefined Roles With Inherited Permissions

- Module introduces predefined role templates (examples):
  - `initiative_member`
  - `goal_owner`
  - `strategy_editor`
  - `risk_coordinator`
- Role templates grant implicit permission bundles.
- User assignment to a template auto-inherits mapped permissions.
- Inheritance is auditable and visible in effective permissions view.

### A7) Explicit Permission Catalog by Domain Object

Permission families are explicitly modeled per object:

- Strategy: `create`, `read`, `edit`, `delete`, `share`, `import`, `export`, `configure`, `build`, `navigate`
- SPM Element Type: `create`, `read`, `edit`, `delete`, `configure_capabilities`, `configure_hierarchy`
- SPM Element (generic): `create`, `read`, `edit`, `delete`, `link`, `unlink`, `share`, `import`, `export`
- KPI: `create`, `read`, `edit`, `delete`, `link`, `unlink`, `import_actuals`, `export`
- Risk: `create`, `read`, `edit`, `delete`, `link`, `unlink`, `mitigate`, `export`
- Team & Membership: `add_member`, `remove_member`, `edit_member_role`, `override_inherited_role`, `grant_custom_permissions`

Permission dependencies:

- `delete` requires `read` + `edit`.
- `share` requires `read`.
- `import`/`export` require `read`.
- `configure_*` permissions require `read` + `edit` on the same object scope.

### A8) Team Membership: Inherited Members + Role Override

- Team panel must show:
  - direct members
  - inherited members (with inheritance source)
- For inherited members, users with manage-members permission can:
  - unlink inherited membership (detach from current element)
  - change inherited role (Viewer/Editor/Admin/Custom)
- If role = `Custom`, permission checklist is shown for per-element grants.
- Effective permissions view must merge:
  - inherited permissions
  - direct role permissions
  - custom overrides

---

## TABLE OF CONTENTS

1. [Overview & Approach](#1-overview--approach)
2. [Core Concepts](#2-core-concepts)
3. [Element Types & Capabilities](#3-element-types--capabilities)
4. [Strategy Configuration](#4-strategy-configuration)
5. [The Graph Model](#5-the-graph-model)
6. [Status & Data Flow](#6-status--data-flow)
7. [KPIs & Risks](#7-kpis--risks)
8. [Vision Alignment](#8-vision-alignment)
9. [Business Rules](#9-business-rules)
10. [Validation Rules](#10-validation-rules)
11. [Calculation Algorithms](#11-calculation-algorithms)
12. [Permissions Model](#12-permissions-model)
13. [Navigation Model](#13-navigation-model)
14. [Technical Data Model](#14-technical-data-model)
15. [API Design](#15-api-design)
16. [Frontend Architecture](#16-frontend-architecture)
17. [Migration Strategy](#17-migration-strategy)

---

## 1. OVERVIEW & APPROACH

### Why This Change

The current SPM module enforces a fixed tree hierarchy: Strategy → Perspective → Goal → Initiative. This does not reflect how organizations actually work. In practice:

- The same Goal may serve multiple Strategies
- An Initiative may contribute to several Goals across different Strategies
- Element types vary by organization — some use "Programs", others use "Pillars" or "Objectives"
- The rigid parent-child model leads to duplicate records, data inconsistency, and reporting gaps

### The New Approach

The SPM module is redesigned around three principles:

**1. Generic Elements with Typed Identity**
Every SPM building block — whether a Perspective, Goal, Initiative, Program, or any custom type — is a single generic entity called an **SPM Element**. Its type is a configuration value defined at the organization level, not hardcoded in the system.

**2. Directed Weighted Graph Structure**
Elements are connected through directed, weighted edges. An element can have multiple children and can participate in multiple strategies. Within a single strategy, an element may have zero, one, or multiple parents as long as no cycle exists. Across strategies, the same element can be reused without duplication. The structure is a **Directed Acyclic Weighted Graph**.

**3. Strategy as a Structured Scope**
A strategy is not a container that owns elements. It defines which element types are used, how they are ordered (the build configuration), and which edges between elements are active in this context. The build configuration is the authoritative guide that governs how elements should be linked within that strategy.

Elements have their own independent identity. Strategies reference and structure them.

### What Does Not Change

- Indicators remain specialized entities with time-series measurement data
- Risks remain specialized entities with assessment and mitigation data
- The Vision module (Pillars, Priorities, Vision Goals) remains fully separate
- Soft deletion is used everywhere — no hard deletes
- All existing business features (budget tracking, progress tracking, team members, workflow) are preserved

---

## 2. CORE CONCEPTS

### SPM Element

A single generic entity representing any strategic building block. Identified by a **type** defined at the organization level. All elements share the same base fields regardless of type.

**Base fields shared by all elements:**

- Name (localized)
- Description (localized)
- Start Date / End Date
- Workflow Status
- Responsible Organization Unit
- Audit trail (created by, modified by, deleted by with timestamps)

Additional fields are unlocked by the element type's declared capabilities. At the organization level these capabilities define the **default starting profile** for the type. When that type is added to a strategy, the defaults are copied into the strategy configuration and may then be overridden for that strategy without changing the organization-level type.

### Element Type

A configuration record at the organization level defining:

- The name of the type (e.g., "Strategic Perspective", "Goal", "Initiative")
- Its default capabilities
- Its visual representation (icon, color)
- Its position in the default display order

Types are not hardcoded. Organizations define their own. A maximum of **6 types per organization** is enforced (4 standard + 2 custom).

An element type does **not** hard-freeze the effective behavior in every strategy. It defines the starting defaults that strategy owners/editors can accept or override when configuring that strategy.

### Strategy

A strategic plan that references a set of elements and defines how they are structured and connected in this context. A strategy owns its **edges** (the connections between elements), its type-level effective settings, and its template lineage (if created from a template), but not the elements themselves. A strategy has its own lifecycle status independent of its elements.

### Strategy Template

A reusable blueprint for creating strategies with common structure and defaults. A template predefines which element types are used, the allowed parent/child combinations, effective per-strategy capability toggles, weight methods, and validation rules. Creating a strategy from a template copies those settings into the new strategy, after which the strategy can diverge.

### Edge (Link)

A directed, weighted connection between two elements within a strategy. An edge says: "Element A contains/directs Element B, with a weight of W in this strategy."

- **Direction**: Parent → Child (top-down, matches how a strategy map is read)
- **Weight**: Contribution of the child to the parent's aggregated metrics
- **Scope**: Every edge belongs to exactly one strategy
- **Multiplicity**: The same child can be linked to multiple parents within the same strategy, provided no cycle is created
- **Root definition**: An element is a root only when it has zero incoming edges in that strategy

---

## 3. ELEMENT TYPES & CAPABILITIES

### Standard Types (expected defaults)

Organizations are expected to configure up to 4 standard types:

1. Perspective
2. Goal
3. Program
4. Initiative

Plus up to 2 custom types.

### Capabilities

Each element type declares its default capabilities. These unlock additional data fields, behaviors, and lifecycle rules, but in strategy setup they act as the **initial value** only. The strategy can override them for that strategy context.

| Capability                | What It Unlocks                                                                     | Notes                                                              |
| ------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Budgetable**            | Planned budget, allocated budget, budget sources, payment disbursement records      | All financial planning and actual spend tracking in one capability |
| **Trackable**             | Progress log entries, milestone entries, own progress + calculated progress display | Enables execution progress tracking                                |
| **Has KPIs**              | KPI tab, KPI link actions, KPI contribution settings                                | Default value copied into strategy type configuration              |
| **Has Risks**             | Risk tab, risk link actions, mitigation workflows                                   | Default value copied into strategy type configuration              |
| **Linkable With Project** | Project-link section and project association actions                                | Can be enabled/disabled per strategy                               |
| **VisionAligned**         | Vision alignment field (Vision Priority, Vision Goals, Vision Indicators)           | Links element to a national/org vision priority                    |
| **Quarterly Measurable**  | Quarterly target/actual tab and threshold-driven alerts                             | Default value copied into strategy type configuration              |

### Universal Features (available to all elements regardless of type)

- **Team Members**: Any element can have team members assigned with roles (owner, editor, viewer)
- **Status Lifecycle**: All elements follow the same status flow regardless of type

### Capability Rules

- **Budgetable** covers both budget planning and payment tracking — there is no separate payments capability
- **Trackable** is independent of Budgetable — progress can be tracked on elements with no budget
- **Budgetable**, **Trackable**, **VisionAligned**, **Has KPIs**, **Has Risks**, **Linkable With Project**, and **Quarterly Measurable** are all strategy-facing effective settings and may differ from one strategy to another for the same element type
- **VisionAligned** remains independent and optional for any type
- Organization-level type capabilities provide defaults; strategy-level configuration stores the effective values used by that strategy

---

## 4. STRATEGY CONFIGURATION & TEMPLATES

### Strategy Creation Modes

New strategies can now be created in one of two ways:

- **From template**: starts from a reusable strategy template with preconfigured type rules, defaults, and weight methods
- **From scratch**: starts as an empty strategy, same as the current flow

Behavior by mode:

- **From scratch**: strategy owners/editors can change all strategy-level type settings, including `budgetable`, `trackable`, `vision_aligned`, `has_kpis`, `has_risks`, `linkable_with_project`, `quarterly_measurable`, root eligibility, parent rules, and edge weighting methods
- **From template**: template-defined type settings are pre-applied and locked during strategy creation. The create flow is used to instantiate the template, not redesign it

### Strategy Templates

A strategy template is an organization-scoped reusable configuration package. It does not contain live elements or live edges, but it defines the build rules and starting defaults for a future strategy.

Each template includes:

- Template name, description, category, and version
- Selected element types
- Effective per-strategy capability defaults for each selected type
- Root eligibility
- Allowed parent type combinations
- Default and pair-level edge weight methods
- Strict hierarchy setting
- Discrepancy threshold
- Optional recommended navigation profile

Template instantiation rule:

- When a strategy is created from a template, the template controls the selected element types and all per-strategy type capability defaults during creation
- The create flow must show these values, but they are not editable there

### Type Level Configuration

Each strategy defines which element types it uses, their allowed parent combinations, and the effective defaults for each type in that strategy.

Example configured type participation:

- Root-capable types: e.g., Perspective
- Types allowed under Perspective: e.g., Goal
- Types allowed under Goal: e.g., Initiative, Program

For every selected type, the strategy also stores effective values such as:

- `is_budgetable`
- `is_trackable`
- `has_kpis`
- `has_risks`
- `linkable_with_project`
- `is_vision_aligned`
- `is_quarterly_measurable`

**This configuration is the authoritative build specification for the strategy.** It defines which element types participate in the strategy, which parent-child type combinations are expected, and what effective capabilities apply to each type in this strategy. Elements within the strategy should be linked following this configuration. The system validates links against the configuration and raises a warning when a link does not conform. Links outside the configured hierarchy are not hard-blocked but are flagged visually and in validation responses so the user consciously decides to proceed.

### Default Weight Input Type

Each strategy also configures the **default weight input type** for all edges within it:

- **PERCENT**: users enter 0–100%. All edges default to percent input.
- **IMPACT**: users enter 1–10. All edges default to impact score input.

This default can be overridden per individual edge when creating or updating a link.

### Strategy Status

A strategy also has its own lifecycle status (see Section 6). The strategy's status gates what operations are allowed on the elements within it.

---

## 5. THE GRAPH MODEL

### Structure

Each element appears **exactly once as a node** within a strategy, but it may have zero, one, or multiple parent edges in that strategy. The same element can also be linked to different strategies, appearing under different parent sets in each.

```
Strategy S1
  ├── Type Level Config [Perspective L1, Goal L2, Initiative L3]
  │   Default weight input: IMPACT
  └── Graph
        ├── Node: Perspective A  [ACTIVE]
        │     ├── Edge (impact: 6) → Node: Goal X  [ACTIVE]
        │     │     └── Edge (impact: 8) → Node: Initiative Alpha  [ACTIVE]
        │     └── Edge (impact: 4) → Node: Goal Y  [ACTIVE]
        │           └── Edge (impact: 5) → Node: Initiative Beta  [DRAFT — excluded from calc]
        └── Node: Perspective B  [ACTIVE]
              └── Edge (impact: 10) → Node: Goal Z  [ACTIVE]

Strategy S2  (reusing Goal Y from S1)
  └── Graph
        └── Node: Perspective X  [ACTIVE]
              └── Edge (impact: 7) → Node: Goal Y  [ACTIVE]  ← same Goal Y record as in S1
```

Key observations:

- Each element appears **once per strategy as a node**
- A node may be linked under multiple parents in the same strategy
- **Goal Y** is the same record used in S1 and S2 with different strategy contexts
- Initiative Beta is in DRAFT status — excluded from parent progress calculations
- Edge weights are independent per edge and per strategy

### Edge Weight

The weight on an edge represents how much the child element contributes to the parent's aggregated metrics.

**Strategy-level default** determines the input mode for all edges in that strategy. **Per-edge override** is available when creating or editing a specific link.

For same-strategy multi-parent support, every edge now carries **two separate semantics**:

- **Parent allocation share**: how much of the child element's owned value is assigned to this parent
- **Parent-local weighting method**: how strongly this child contributes relative to its siblings under that parent

These are different on purpose:

- Allocation share prevents double counting when one child is used under multiple parents
- Parent-local weighting controls rollup behavior among siblings under a single parent

**Supported weighting methods per edge:**

- **PERCENT**: users enter 0–100%. Stored as decimal (e.g., 60% → 0.6000)
- **IMPACT**: users enter 1–10. Normalized relative to siblings under the same parent
- **EQUAL**: no manual value; equal sibling contribution under that parent
- **BUSINESS_VALUE**: derived from child `business_value`, normalized relative to siblings under the same parent
- **BUDGETABLE**: derived from child `planned_budget`, normalized relative to siblings under the same parent

**Parent allocation rule:**

- If a child has only one parent in the strategy, allocation defaults to `100%`
- If a child has multiple parents in the same strategy, the sum of `allocation_percent` across all active incoming edges for that child must equal `100%`
- Propagation uses `allocation_percent × parent_local_effective_weight`

### Shared Elements (Cross-Strategy Reuse)

When the same element is used in multiple strategies, it is marked in the element library and on its detail page with a **"used in X strategies" indicator** listing those strategies. Within a single strategy, each element still appears exactly once as a node — there is no within-strategy duplication of the record even if it has multiple parents. The element's global data (name, description, status) is the same in all strategies; only the strategy context (parents, weights, effective type settings, navigation placement) differs per strategy.

---

## 6. STATUS & DATA FLOW

This section defines the complete lifecycle of every SPM element and strategy — how they progress from creation to completion, what each status means operationally, and exactly what changes apply to data and calculations at each transition.

---

### 6.1 Element Status Lifecycle

#### Status Definitions

| Status          | Meaning                                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| **DRAFT**       | Element is being prepared. Not yet operational. Editable freely. Excluded from parent calculations.                  |
| **ACTIVE**      | Element is live and operational. Tracking has begun. Included in all calculations.                                   |
| **ON_HOLD**     | Element is temporarily paused. Progress frozen at last value. Still counted in parent calculations at frozen value.  |
| **COMPLETED**   | Element finished successfully. All data frozen. Contributes final progress to parents permanently.                   |
| **DEACTIVATED** | Element suspended indefinitely. Excluded from all calculations. Can be reactivated.                                  |
| **DEPRECATED**  | Element is obsolete. Permanently excluded from all calculations. Cannot be linked to new strategies. Terminal state. |

#### Allowed Status Transitions (TODO :: ADD WHO CAN DO THAT AND POST-CONDITION and PRE-CONDITION) (STATE DIAGRAM) (Permission Model, Build and Validate)

```
DRAFT ──────→ ACTIVE ──────→ ON_HOLD ──────→ ACTIVE (resume)
  │               │               │
  │               │               └──────→ DEACTIVATED ──────→ DEPRECATED
  │               │                               ↑
  │               ├──────→ COMPLETED ──────→ DEPRECATED
  │               │
  │               └──────→ DEACTIVATED
  │
  └──────→ DEPRECATED (cancelled before ever being activated)
```

| From        | To          | Description                              |
| ----------- | ----------- | ---------------------------------------- |
| DRAFT       | ACTIVE      | Element activated and made operational   |
| DRAFT       | DEPRECATED  | Element cancelled before activation      |
| ACTIVE      | ON_HOLD     | Element temporarily paused               |
| ACTIVE      | COMPLETED   | Element marked as successfully finished  |
| ACTIVE      | DEACTIVATED | Element suspended indefinitely           |
| ON_HOLD     | ACTIVE      | Element resumed from hold                |
| ON_HOLD     | DEACTIVATED | Element suspended while on hold          |
| COMPLETED   | DEPRECATED  | Completed element archived permanently   |
| DEACTIVATED | ACTIVE      | Element reactivated                      |
| DEACTIVATED | DEPRECATED  | Deactivated element archived permanently |

**Forbidden transitions** (any not listed above is rejected):

- COMPLETED → ACTIVE (cannot reverse completion)
- DEPRECATED → any status (no exit from terminal state)
- ON_HOLD → COMPLETED (must resume to ACTIVE first)

---

### 6.2 What Changes at Each Status Transition

**DRAFT → ACTIVE**

The element is now live. This is the most significant transition.

| Area                    | Change                                                                                                                                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | Start date is locked — cannot be changed after activation. Status becomes globally ACTIVE across all strategies containing this element.                                                                                    |
| **Budgetable**          | Planned budget locked for currency type. Allocated budget and payments can still be updated. Budget fields become visible in financial dashboards and parent budget aggregations.                                           |
| **Trackable**           | Progress tracking begins. Progress log entries can now be created. Milestones become actionable. Expected progress calculation begins from start date. Both `own_progress` and `calculated_progress` fields are now active. |
| **VisionAligned**       | Vision alignment becomes visible in Vision module alignment reports.                                                                                                                                                        |
| **Indicators linked**   | Target tracking begins. Actuals are now measured against targets. Indicator contributions to health score are active.                                                                                                       |
| **Risks linked**        | Risk monitoring starts. Risk phase progression tracking begins. Risk contribution to risk score is now active.                                                                                                              |
| **Calculations**        | Element is included in all parent progress calculations in every strategy it belongs to. Affected parent elements recalculate upward through the graph.                                                                     |
| **Real-time UI update** | If the same screen is displaying parent elements or strategy-level summaries, a real-time update is pushed via WebSocket/SSE so the displayed aggregated values refresh without requiring a manual page reload.             |
| **Notifications**       | Team members notified. Parent element owners notified.                                                                                                                                                                      |

---

**ACTIVE → ON_HOLD**

| Area                    | Change                                                                                                                                       |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | Description and end date extension are editable. All other fields locked.                                                                    |
| **Budgetable**          | No new planned budget changes. Allocated budget adjustments for committed amounts allowed. Payments for pre-committed amounts still allowed. |
| **Trackable**           | No new progress log entries. Both `own_progress` and `calculated_progress` **frozen at last recorded values**. Milestones suspended.         |
| **Indicators linked**   | Actuals can still be entered for monitoring continuity. Targets unchanged.                                                                   |
| **Risks linked**        | Risk monitoring continues. Risk phase progression continues.                                                                                 |
| **Calculations**        | Element remains in parent calculations using frozen progress value. Parents are not required to recalculate unless other siblings change.    |
| **Real-time UI update** | Parent progress values update to reflect frozen state.                                                                                       |
| **Notifications**       | Team members and parent element owners notified.                                                                                             |

---

**ON_HOLD → ACTIVE (Resume)**

| Area                    | Change                                                                                                                |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | All editing rights restored to ACTIVE rules.                                                                          |
| **Trackable**           | Progress tracking resumes. New entries can be created. Expected progress recalculates accounting for the hold period. |
| **Calculations**        | Element rejoins active weighted calculations. Parent elements recalculate immediately.                                |
| **Real-time UI update** | Parent progress summaries refresh.                                                                                    |
| **Notifications**       | Team members and parent owners notified of resumption.                                                                |

---

**ACTIVE → COMPLETED**

| Area                    | Change                                                                                                                                                                                 |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | All fields locked (read-only). End date set to today if not already provided.                                                                                                          |
| **Budgetable**          | No budget changes. Final payment reconciliation allowed within a 30-day grace period. After grace period all payments locked.                                                          |
| **Trackable**           | `own_progress` fixed at 100% unless the last logged entry is lower, in which case fixed at that value. `calculated_progress` retains its last computed value. No new progress entries. |
| **Indicators linked**   | Final period actuals can still be recorded for the reporting period. No new actuals after reporting period closes.                                                                     |
| **Risks linked**        | Open risk phases must be resolved, transferred, or accepted. No new risk assessments.                                                                                                  |
| **Calculations**        | Element contributes its final frozen progress to parent calculations permanently.                                                                                                      |
| **Real-time UI update** | Parent progress summaries refresh to reflect final value.                                                                                                                              |
| **Notifications**       | Team members and parent owners notified.                                                                                                                                               |

---

**ACTIVE → DEACTIVATED**

| Area                    | Change                                                                                                                                                                                        |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | All fields immediately read-only.                                                                                                                                                             |
| **Budgetable**          | No new budget changes. No new payments. Existing records retained.                                                                                                                            |
| **Trackable**           | Progress tracking stops. Both progress values frozen. No new entries.                                                                                                                         |
| **Indicators linked**   | No new actuals. Indicators remain linked for historical record only.                                                                                                                          |
| **Risks linked**        | Risk monitoring paused. Open risk phases suspended.                                                                                                                                           |
| **Calculations**        | Element **removed from all parent calculations**. Its edge weight is treated as zero. Affected parent elements recalculate upward through the graph.                                          |
| **Children**            | Children of this element are **not automatically deactivated**. A warning is shown listing all children and all parent elements in all strategies that will be affected by the recalculation. |
| **Real-time UI update** | Parent progress and budget summaries refresh immediately via push notification to any currently open screens showing affected parents.                                                        |
| **Notifications**       | Team members and all parent element owners in all strategies notified.                                                                                                                        |

---

**DEACTIVATED → ACTIVE (Reactivate)**

| Area                    | Change                                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | Editing rights restored to ACTIVE rules.                                                                                        |
| **Trackable**           | Progress tracking resumes. Gap period has no entries. Expected progress calculation adjusts to account for the deactivated gap. |
| **Calculations**        | Element re-enters all parent calculations. Affected parents recalculate.                                                        |
| **Real-time UI update** | Parent summaries refresh.                                                                                                       |
| **Notifications**       | Team members and parent owners notified.                                                                                        |

---

**Any Status → DEPRECATED**

Terminal state. No return.

| Area                    | Change                                                                                                                                             |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements**        | Permanently read-only.                                                                                                                             |
| **Calculations**        | Excluded from all calculations (same as DEACTIVATED). Affected parents recalculate.                                                                |
| **Linking**             | Cannot be linked to any new strategy. Cannot receive new indicator or risk links.                                                                  |
| **Visibility**          | Hidden from all default views (strategy maps, dashboards, active element library). Visible only in archive/history views when explicitly filtered. |
| **Existing links**      | All existing strategy edges retained in database for historical integrity. Calculation engine ignores them.                                        |
| **Real-time UI update** | Parent summaries refresh.                                                                                                                          |
| **Notifications**       | Team members notified.                                                                                                                             |

---

### 6.3 Strategy Status Lifecycle

#### Strategy Status Definitions

| Status        | Meaning                                                                                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **DRAFT**     | Strategy is being configured. Type configuration and build rules can be set. Elements can be added and linked. Progress and payments cannot be recorded. |
| **ACTIVE**    | Strategy is live. All tracking, calculations, and reporting are operational.                                                                             |
| **SUSPENDED** | Strategy is temporarily paused (e.g., strategic review). No new tracking. Existing data read-only within this strategy context.                          |
| **COMPLETED** | Strategy period ended. All data frozen in this context. Historical access only.                                                                          |
| **ARCHIVED**  | Strategy permanently archived. Read-only historical reference. Terminal state.                                                                           |

#### Allowed Strategy Transitions

```
DRAFT ──────→ ACTIVE ──────→ SUSPENDED ──────→ ACTIVE (resume)
  │               │
  │               ├──────→ COMPLETED ──────→ ARCHIVED
  │               └──────→ ARCHIVED (abandoned)
  │
  └──────→ ARCHIVED (abandoned before activation)
```

#### How Strategy Status Affects Element Operations

| Strategy Status | Element Operations Allowed                                                            |
| --------------- | ------------------------------------------------------------------------------------- |
| **DRAFT**       | Add/remove elements and links. Edit element data. Cannot record progress or payments. |
| **ACTIVE**      | All operations allowed per each element's own status rules.                           |
| **SUSPENDED**   | Read-only. No new progress entries or payments. No new element or link additions.     |
| **COMPLETED**   | Read-only. No changes to any element within this strategy context.                    |
| **ARCHIVED**    | Read-only historical access. No changes.                                              |

---

### 6.4 Data Flow Summary by Capability

#### If Element is BUDGETABLE:

| Operation                             | DRAFT | ACTIVE | ON_HOLD            | COMPLETED        | DEACTIVATED | DEPRECATED |
| ------------------------------------- | ----- | ------ | ------------------ | ---------------- | ----------- | ---------- |
| Set planned budget                    | ✅     | ✅      | ❌                  | ❌                | ❌           | ❌          |
| Update allocated budget               | ✅     | ✅      | ✅ (committed only) | ❌                | ❌           | ❌          |
| Add budget source                     | ✅     | ✅      | ❌                  | ❌                | ❌           | ❌          |
| Create payment record                 | ❌     | ✅      | ✅ (committed only) | ✅ (grace period) | ❌           | ❌          |
| Edit payment                          | ❌     | ✅      | ✅                  | ✅ (grace period) | ❌           | ❌          |
| View budget & payments                | ✅     | ✅      | ✅                  | ✅                | ✅           | ✅          |
| Included in parent budget aggregation | ❌     | ✅      | ✅                  | ✅                | ❌           | ❌          |

#### If Element is TRACKABLE:

| Operation                               | DRAFT | ACTIVE | ON_HOLD    | COMPLETED  | DEACTIVATED | DEPRECATED |
| --------------------------------------- | ----- | ------ | ---------- | ---------- | ----------- | ---------- |
| Add progress log entry                  | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| Add milestone                           | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| Mark milestone complete                 | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| View own_progress                       | ✅     | ✅      | ✅          | ✅          | ✅           | ✅          |
| View calculated_progress                | ✅     | ✅      | ✅          | ✅          | ✅           | ✅          |
| View discrepancy indicator              | —     | ✅      | ✅          | ✅          | —           | —          |
| Included in parent progress calculation | ❌     | ✅      | ✅ (frozen) | ✅ (frozen) | ❌           | ❌          |

#### For INDICATORS linked to element (optional):

| Operation                             | DRAFT | ACTIVE | ON_HOLD        | COMPLETED            | DEACTIVATED | DEPRECATED |
| ------------------------------------- | ----- | ------ | -------------- | -------------------- | ----------- | ---------- |
| Link new indicator                    | ✅     | ✅      | ❌              | ❌                    | ❌           | ❌          |
| Unlink indicator                      | ✅     | ✅      | ❌              | ❌                    | ❌           | ❌          |
| Enter actual values for indicator     | ❌     | ✅      | ✅ (monitoring) | ✅ (reporting period) | ❌           | ❌          |
| Indicator contributes to health score | ❌     | ✅      | ✅              | ✅                    | ❌           | ❌          |

#### For RISKS linked to element (optional):

| Operation                      | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED | DEPRECATED |
| ------------------------------ | ----- | ------ | ------- | --------- | ----------- | ---------- |
| Link new risk                  | ✅     | ✅      | ❌       | ❌         | ❌           | ❌          |
| Unlink risk                    | ✅     | ✅      | ❌       | ❌         | ❌           | ❌          |
| Progress risk phase            | ❌     | ✅      | ✅       | ❌         | ❌           | ❌          |
| Risk contributes to risk score | ❌     | ✅      | ✅       | ✅         | ❌           | ❌          |

#### For TEAM MEMBERS (all elements):

| Operation          | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED | DEPRECATED |
| ------------------ | ----- | ------ | ------- | --------- | ----------- | ---------- |
| Add team member    | ✅     | ✅      | ✅       | ❌         | ❌           | ❌          |
| Remove team member | ✅     | ✅      | ✅       | ❌         | ❌           | ❌          |
| View team members  | ✅     | ✅      | ✅       | ✅         | ✅           | ✅          |

#### For GRAPH LINKS (edges in a strategy):

| Operation                   | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED    | DEPRECATED |
| --------------------------- | ----- | ------ | ------- | --------- | -------------- | ---------- |
| Add as child to a parent    | ✅     | ✅      | ❌       | ❌         | ❌              | ❌          |
| Remove from parent (unlink) | ✅     | ✅      | ❌       | ❌         | ✅ (admin only) | ❌          |
| Update edge weight          | ✅     | ✅      | ❌       | ❌         | ❌              | ❌          |

---

### 6.5 Status Transition — Permission Requirements

All status transitions are governed by permissions managed through **OpenFGA**. Each role assigned to a user on an element, strategy, or module is internally translated into a set of OpenFGA permissions. The rules below are expressed in terms of required permissions; the specific roles that carry these permissions are defined in the permissions model (Section 12).

| Transition                  | Required Permission                                                                                       |
| --------------------------- | --------------------------------------------------------------------------------------------------------- |
| DRAFT → ACTIVE              | `spm_element:activate` (held by element owner or element editor)                                          |
| DRAFT → DEPRECATED          | `spm_element:deprecate` (held by element owner or module admin)                                           |
| ACTIVE → ON_HOLD            | `spm_element:hold` (held by element owner, element editor, or strategy editor of any containing strategy) |
| ACTIVE → COMPLETED          | `spm_element:complete` (held by element owner or element editor)                                          |
| ACTIVE → DEACTIVATED        | `spm_element:deactivate` (held by element owner or module admin)                                          |
| ON_HOLD → ACTIVE            | `spm_element:resume` (held by element owner or element editor)                                            |
| ON_HOLD → DEACTIVATED       | `spm_element:deactivate` (held by element owner or module admin)                                          |
| COMPLETED → DEPRECATED      | `spm_element:deprecate` (held by module admin or strategy owner of any containing strategy)               |
| DEACTIVATED → ACTIVE        | `spm_element:reactivate` (held by element owner or module admin)                                          |
| DEACTIVATED → DEPRECATED    | `spm_element:deprecate` (held by module admin)                                                            |
| Strategy DRAFT → ACTIVE     | `strategy:activate` (held by strategy owner or strategy editor)                                           |
| Strategy ACTIVE → SUSPENDED | `strategy:suspend` (held by strategy owner or module admin)                                               |
| Strategy ACTIVE → COMPLETED | `strategy:complete` (held by strategy owner or module admin)                                              |
| Strategy SUSPENDED → ACTIVE | `strategy:resume` (held by strategy owner or module admin)                                                |
| Strategy ANY → ARCHIVED     | `strategy:archive` (held by module admin)                                                                 |

---

### 6.6 Status Propagation to Parent Calculations

When an element's status changes, all parent elements in every strategy that contains it must recalculate their derived metrics. This propagation traverses all the way to the root of each affected graph.

```
Status change on Element E
  → Collect all strategies where E appears as a linked child
    → For each strategy S:
      → Find all ancestors of E in S (traverse upward to root)
        → Recalculate calculated_progress for each ancestor
        → Recalculate health score for each ancestor (if indicators linked)
        → Recalculate budget aggregation for each ancestor (if budgetable)
```

**Recalculation execution:** Triggered asynchronously to avoid blocking the user's status-change action.

**Real-time UI consideration:** When the user who triggered the status change (or any other user) has a screen open that is displaying affected parent elements or strategy-level summaries, the system pushes real-time updates via WebSocket or Server-Sent Events (SSE) to refresh those displayed values without requiring a page reload. For screens that do not maintain a live connection, a short polling fallback (every 5–10 seconds) is applied while the page is open and the strategy is in ACTIVE state.

---

## 7. KPIs & RISKS

Indicators and Risks are **specialized entities**, not generic SPM elements. They have rich domain-specific data that does not fit the generic element model.

### KPIs

Remain fully specialized with:

- Time-series measurement data (baseline, target, actual by Year-Quarter)
- Measure type (Monetary, Quantity, Rate)
- Indicator type (Key Indicator, Vision Indicator, Sub-Indicator, Benchmarking, Cross-Cutting, Vertical Alignment)
- Bigger-is-better flag
- Absolute baseline and absolute target
- Team member management

**Lifecycle**: Indicators have their own status (DRAFT, ACTIVE, DEPRECATED).

**Linking to elements**: Linking an Indicator to an SPM element is **entirely optional**. An indicator can exist as a standalone entity with no element links. When linked, the link is strategy-scoped and carries a weight representing the indicator's contribution to that element's health score. The edge is typed as **"measures"**.

### Risks

Remain fully specialized with:

- Risk category, type, urgency, escalation level
- Impact assessment (base and residual impact and probability)
- Mitigation plan with treatment decision and costs
- Mitigation phases with status and progress tracking
- Monitoring frequency and owner assignment
- Team member management

**Lifecycle**: Risks have their own status flow (IDENTIFIED, ASSESSED, MITIGATING, RESOLVED, ACCEPTED, CLOSED).

**Linking to elements**: Linking a Risk to an SPM element is **entirely optional**. A risk can exist without being linked to any element. When linked, the link is strategy-scoped. The edge is typed as **"threatens"**.

### Scope of Linking (Phased Rollout)

| Link Type                            | Phase 1 (Initial Release)        | Future Release |
| ------------------------------------ | -------------------------------- | -------------- |
| Indicator → Goal-type element        | ✅ Frontend enabled               | —              |
| Indicator → Initiative-type element  | ✅ Frontend enabled               | —              |
| Indicator → Perspective-type element | Backend built, Frontend disabled | ✅              |
| Indicator → Strategy (top-level KPI) | Backend built, Frontend disabled | ✅              |
| Risk → Goal-type element             | ✅ Frontend enabled               | —              |
| Risk → Initiative-type element       | ✅ Frontend enabled               | —              |
| Risk → Perspective-type element      | Backend built, Frontend disabled | ✅              |
| Risk → Strategy (top-level)          | Backend built, Frontend disabled | ✅              |

---

## 8. VISION ALIGNMENT

The Vision module (Vision Pillar → Vision Priority → Vision Goal / Vision Indicator) remains completely separate and unchanged.

SPM Elements with the **VisionAligned** capability have an optional **Vision Alignment** field. This is metadata — it shows which vision priority this element serves and does not create a structural dependency between the SPM graph and the Vision hierarchy.

Optional alignment fields per element:

- Vision Priority (select one from the Vision module)
- One or more Vision Goals within the selected Vision Priority
- One or more Vision Indicators within the selected Vision Priority

Vision alignment data is read-only in Vision module reports and can only be edited from the SPM element's detail page.

---

## 9. BUSINESS RULES

### Element Ownership & Identity

**BR-01** An element belongs to an organization. Elements cannot be shared across organizations.

**BR-02** The creator of an element is its **initial owner**. Ownership can be transferred: any team member with `element:manage_members` permission can assign the **owner role** to another user via the team membership panel. At any time, there is exactly one owner per element.

**BR-03** An element has a single global identity. Editing its name, description, dates, or status changes it everywhere it is referenced — across all strategies that contain it.

**BR-04** An element can exist without being part of any strategy (orphan element in the library) and can be linked to strategies later.

**BR-05** An element can be part of multiple strategies simultaneously.

### Linking Rules

**BR-06** An element is added to a strategy either by creating an edge involving it or by explicitly adding it as a root element with no parent.

**BR-07** Within a single strategy, each element appears exactly once as a node. It may be a root node (zero parents), have one parent, or have multiple parents in the same strategy.

**BR-08** The same element can be referenced in multiple strategies. In each strategy it has its own parent set, its own edge weights, and its own display order. The element's global data remains identical across all strategies.

**BR-09** Edges are strategy-scoped. When a strategy is soft-deleted, all its edges are removed. The elements themselves are unaffected — they remain in the organization's element library as orphan elements with no active strategy context. These elements have no operational impact until linked to another strategy.

**BR-10** Linking an existing element from another strategy does not change that element's data or its links in the other strategy.

**BR-11** A DEPRECATED element cannot be linked to any new strategy or receive new indicator/risk links.

**BR-12** Edges can only be created or modified when both the strategy and the target element are in an editable state (not COMPLETED, ARCHIVED, or DEPRECATED).

**BR-13** Links within a strategy should follow the strategy's type level configuration. A link that does not match the configured hierarchy produces a warning but is not hard-blocked.

### Deletion Rules

**BR-14** All deletions are soft deletes. No record is permanently removed.

**BR-15** Soft-deleting an element sets `deleted_at` on the element record. All strategy edges remain in the database but the element is excluded from all views and calculations.

**BR-16** Removing an element from a strategy removes only the edge(s) connecting it in that strategy. The element itself is unaffected.

**BR-17** Soft-deleting a strategy marks the strategy record only. All linked elements are unaffected.

**BR-18** A soft-deleted element behaves identically to DEPRECATED for calculation purposes — excluded from all parent metrics.

**BR-19** Restoring a soft-deleted element (clearing `deleted_at`) makes it visible again in all strategies it was linked to. Requires `element:restore` permission (held by module admin or strategy owner of any containing strategy).

### Status Rules

**BR-20** Workflow status is a global property of the element. The same status applies in all strategies.

**BR-21** Status transitions must follow the allowed map in Section 6.1. Any other transition is rejected.

**BR-22** Transitioning an element to DEACTIVATED or DEPRECATED does not automatically change the status of its children. Children continue independently.

**BR-23** When deactivating an element, the system presents a warning listing all parent elements across all strategies that will be affected by the recalculation.

**BR-24** DEPRECATED is a terminal state. No transition out of DEPRECATED is possible for any user.

**BR-25** A strategy cannot be activated (DRAFT → ACTIVE) if it has no type level configuration or no elements.

**BR-26** A strategy's COMPLETED or ARCHIVED transition does not change the status of the elements within it.

### Budget Rules

**BR-27** Planned and allocated budget are global properties of an element. They do not vary per strategy.

**BR-28** Budget fields are available only on elements whose type has the Budgetable capability.

**BR-29** Budget aggregation at parent level is context-computed: sums budgets of all descendant elements reachable through that strategy's graph, excluding DEACTIVATED and DEPRECATED elements.

**BR-30** Since an element may now be linked to multiple parents in the same strategy, budget and progress propagation must use parent-allocation shares so that the total propagated value of a child across all its parents never exceeds the child's owned value.

**BR-31** If an element has multiple active parents in the same strategy, the sum of its active incoming `allocation_percent` values must equal `100%`.

**BR-32** An element's consumed budget equals the sum of all its payment records globally across all strategies.

### Progress Rules

**BR-33** Progress can only be logged on elements whose type has the Trackable capability and whose status is ACTIVE.

**BR-34** Every Trackable element maintains two progress values:

- **own_progress**: the rate from the most recent manually entered progress log entry
- **calculated_progress**: derived from the element's active children's weighted progress in a given strategy context

**BR-35** For leaf elements (no active children in the strategy), `calculated_progress` is not applicable. Only `own_progress` is shown.

**BR-36** For parent elements (with active children), both values exist. `calculated_progress` is used for upward propagation. If the two values differ by more than a configurable threshold (default 5%), a discrepancy indicator is shown to alert the owner.

**BR-37** DRAFT and DEACTIVATED and DEPRECATED children are excluded from parent `calculated_progress`. Their edge weight is not counted in the denominator.

**BR-38** ON_HOLD and COMPLETED children are included using their frozen last value.

**BR-39** Parent-local edge weight defaults to equal distribution among all included siblings under the same parent if no weight method input is set.

**BR-40** When a child has multiple parents, upward propagation to each parent uses the child's owned value multiplied by that edge's `allocation_percent`, then applies the selected edge weighting method inside the target parent's sibling set.

---

## 10. VALIDATION RULES

### Graph Integrity

**VR-01 Cycle Prevention — UI Layer**
When selecting a parent for an element, the parent picker excludes the element itself and all its existing descendants in the current strategy. These are filtered before the picker renders.

**VR-02 Cycle Prevention — Service Layer**
Before persisting a new edge, the service performs a reachability check across the current strategy graph. If the proposed parent is reachable from the proposed child through any existing path, the request is rejected with `GRAPH_CYCLE_DETECTED`.

**VR-03 Duplicate Edge Prevention**
An element may have multiple parents within the same strategy, but the exact same edge pair cannot be created twice. Attempting to create a duplicate edge with the same `strategy_id`, `parent_element_id`, and `child_element_id` is rejected with `DUPLICATE_STRATEGY_EDGE`.

**VR-04 Self-Link**
An element cannot be linked to itself. Rejected by database CHECK constraint and service layer.

**VR-05 Cross-Organization Link**
An edge cannot connect elements from different organizations. Service validates that both elements share the same `organization_id`.

**VR-06 Type Hierarchy Conformance**
When creating an edge, the service validates that the parent element's type and child element's type conform to the strategy's configured type levels. If they do not conform, the response includes a `HIERARCHY_WARNING` field. The link is created but the caller is informed. The UI surfaces this as a visual warning on the edge.

### Weight Validation

**VR-07** Impact Score input must be an integer between 1 and 10 inclusive.

**VR-08** Percentage input must be between 0 and 100 inclusive. Stored as decimal (0.0–1.0).

**VR-09** `allocation_percent` must be between 0 and 100 inclusive. Stored as decimal (0.0–1.0). For children with a single active parent, the value defaults to 1.0.

**VR-09A** For any child with multiple active parents in the same strategy, the sum of active incoming `allocation_percent` values must equal exactly 1.0 (within a small configurable rounding tolerance).

**VR-09B** Edge weighting method must be one of: `PERCENT`, `IMPACT`, `EQUAL`, `BUSINESS_VALUE`, `BUDGETABLE`.

**VR-09C** Manual `weight_value` is optional only for methods that require manual input. It is ignored for `EQUAL`, `BUSINESS_VALUE`, and `BUDGETABLE`.

### Element Type Limits

**VR-10** An organization may not define more than 6 element types.

**VR-11** At least one element type must be defined before a strategy can be created.

**VR-12** An element type cannot be deleted if any element of that type exists (including soft-deleted elements).

**VR-13** An element type's code must be unique within the organization.

### Element Field Rules

**VR-14** Element name is required and must not be empty in at least the default organization language.

**VR-15** End date, if provided, must be on or after start date.

**VR-16** Organization unit, if provided, must reference a valid and active unit.

**VR-17** An element's type cannot be changed after the element is created.

**VR-18** Start date cannot be changed after the element has been activated (ACTIVE or beyond).

### Status Transition Validation

**VR-19** Any status transition not listed in the allowed map (Section 6.1) is rejected with `INVALID_STATUS_TRANSITION`.

**VR-20** A DEPRECATED element cannot have its status changed under any circumstances.

**VR-21** Before activating an element (DRAFT → ACTIVE):

- Name must be set in the default organization language
- If Trackable: start date must be set
- If Budgetable: planned budget is recommended (warning shown if missing, not a hard block)
- If VisionAligned: alignment is optional (no block)

**VR-22** Before completing an element (ACTIVE → COMPLETED):

- If Trackable: at least one progress log entry must exist
- If Budgetable: alert shown if consumed budget < allocated budget (under-spend notice, not a block)

**VR-23** Before deprecating an element:

- Element must be in DRAFT, COMPLETED, or DEACTIVATED status. Cannot deprecate an ACTIVE or ON_HOLD element directly.

**VR-24** A strategy cannot transition from DRAFT to ACTIVE without at least one element and a type level configuration.

### Budget Validation

**VR-25** Allocated budget exceeding planned budget shows a warning. Not a hard block.

**VR-26** Payment amount must be positive and non-zero.

**VR-27** Budget and payment fields can only be set on elements whose type has the Budgetable capability. Otherwise rejected with `CAPABILITY_NOT_SUPPORTED`.

### Progress Validation

**VR-28** Progress rate must be between 0 and 100 inclusive.

**VR-29** Progress entries can only be created on elements with Trackable capability and ACTIVE status.

**VR-30** Progress entry date must not be in the future.

**VR-31** Milestone date should fall within element's start and end date range. Warning shown if outside (not a hard block).

### Strategy Configuration Validation

**VR-32** Type level references must belong to the same organization as the strategy.

**VR-33** Each selected element type may appear only once in a strategy's type configuration.

**VR-34** Level order values must be unique per strategy.

**VR-35** Default weight input type must be one of: PERCENT, IMPACT, EQUAL, BUSINESS_VALUE, BUDGETABLE.

**VR-35A** Strategy effective capability values must be stored per selected type and initialized from the organization-level element type defaults.

**VR-35B** `linkable_with_project`, `has_kpis`, and `has_risks` may be toggled per strategy even when the organization-level type default differs.

**VR-35C** A strategy template may not reference deleted element types or element types from another organization.

### Indicator & Risk Link Validation

**VR-36** An indicator cannot be linked to the same element more than once in the same strategy.

**VR-37** A risk cannot be linked to the same element more than once in the same strategy.

**VR-38** Indicator and risk links can only be created when the strategy is DRAFT or ACTIVE.

**VR-39** Indicator and risk links cannot be created on elements in COMPLETED, DEACTIVATED, or DEPRECATED status.

---

## 11. CALCULATION ALGORITHMS

### 11.1 Progress Calculation

Progress is calculated on a DAG, not a tree. Because the same element may sit under multiple parents in the same strategy, bottom-up propagation must split that child's owned value across its incoming parent links before each parent performs its local sibling weighting. Status determines inclusion. Two values are maintained for Trackable elements: `own_progress` (manual) and `calculated_progress` (derived).

```
FUNCTION calculateProgress(elementId, strategyId):

  childLinks = QUERY direct outgoing edges for elementId in strategyId
               joined with child workflow status

  includedLinks = childLinks WHERE child.status IN ('ACTIVE', 'ON_HOLD', 'COMPLETED')

  IF includedLinks IS EMPTY:
    latestEntry = most recent spm_element_progress record for elementId
    RETURN latestEntry.rate IF exists ELSE 0

  normalizedSiblingWeights = NORMALIZE_WITHIN_PARENT(includedLinks)
  weightedSum = 0
  totalWeight = 0

  FOR EACH link IN normalizedSiblingWeights:
    childValue = calculateProgress(link.child_element_id, strategyId)
    childParentCount = COUNT_ACTIVE_PARENTS(link.child_element_id, strategyId)
    allocation = link.allocation_percent

    IF childParentCount <= 1 AND allocation IS NULL:
      allocation = 1.0

    propagatedValue = childValue × allocation
    localWeight = link.effective_parent_weight

    weightedSum += propagatedValue × localWeight
    totalWeight += localWeight

  IF totalWeight = 0:
    RETURN 0

  RETURN weightedSum / totalWeight
```

**own_progress vs calculated_progress:**

- `own_progress` = latest rate from `spm_element_progress` table (leaf nodes: this is the only progress value)
- `calculated_progress` = result of the algorithm above (parent nodes: this is the authoritative value propagated upward)
- For parent elements: if `|own_progress - calculated_progress| > discrepancy_threshold` (configurable, default 5%), a discrepancy flag is set and displayed in the UI as a prompt for the element owner to review
- A shared child does **not** contribute its full progress independently to every parent
- Instead, its progress is first split by incoming `allocation_percent`, then locally weighted among each parent's siblings

### 11.1A Why Progress Needs Both Allocation And Weight

For progress rollup, two different questions must be answered on each link:

- **Allocation**: how much of the child belongs to this parent when the child has multiple parents
- **Weight**: how important this child is relative to its siblings under this one parent

These are intentionally separate because they normalize over different groups:

- **Allocation** is normalized across a single child's incoming parent links
- **Weight** is normalized across a single parent's outgoing child links

If one field is used for both meanings, the graph becomes logically inconsistent as soon as a shared child also has siblings.

#### Example: Why `contribution_percent` Alone Is Not Enough

Assume:

- `I1` progress = `80%`
- `I2` progress = `50%`
- `I3` progress = `40%`

Links:

- `I1 -> Goal X`
- `I1 -> Goal Y`
- `I2 -> Goal X`
- `I3 -> Goal Y`

Now suppose the only field on each link is `contribution_percent`:

- `I1 -> Goal X = 70%`
- `I1 -> Goal Y = 30%`
- `I2 -> Goal X = 30%`
- `I3 -> Goal Y = 70%`

At first glance this looks valid, but the field meaning becomes ambiguous.

**Interpretation A: contribution_percent means cross-parent split**

Then for `I1`:

- Goal X receives `80 × 70% = 56`
- Goal Y receives `80 × 30% = 24`

That is fine.

But now for `I2 -> Goal X = 30%`:

- Goal X receives `50 × 30% = 15`

Where does the remaining `35` go? `I2` has no other parent in this example, so part of the child disappears.

Likewise for `I3 -> Goal Y = 70%`:

- Goal Y receives `40 × 70% = 28`

Where does the remaining `12` go?

So if `contribution_percent` is treated as allocation, every child's incoming parent-link percentages must sum to `100%`. That is a cross-parent rule, not a sibling-weight rule.

**Interpretation B: contribution_percent means sibling importance within each parent**

Then under Goal X:

- `I1 = 70%`
- `I2 = 30%`
- Goal X progress = `(80 × 70%) + (50 × 30%) = 56 + 15 = 71%`

Under Goal Y:

- `I1 = 30%`
- `I3 = 70%`
- Goal Y progress = `(80 × 30%) + (40 × 70%) = 24 + 28 = 52%`

This seems valid locally, but now change Goal X sibling importance only:

- `I1 -> Goal X = 40%`
- `I2 -> Goal X = 60%`

Keep Goal Y unchanged:

- `I1 -> Goal Y = 30%`
- `I3 -> Goal Y = 70%`

Now `I1` contributes:

- to Goal X: `80 × 40% = 32`
- to Goal Y: `80 × 30% = 24`
- total reused from `I1` = `56`

Part of the child's progress vanished simply because Goal X changed its local sibling weighting.

If Goal X instead sets `I1 = 80%` while Goal Y stays at `30%`:

- to Goal X: `80 × 80% = 64`
- to Goal Y: `80 × 30% = 24`
- total reused from `I1` = `88`

Now the same child contributes more than its owned progress.

This is the core contradiction: a single `contribution_percent` field cannot safely satisfy both:

- the **child-centric rule**: percentages across one child's parents should sum to `100%`
- the **parent-centric rule**: percentages across one parent's children should sum to `100%`

#### Correct Model

The model therefore separates the two concerns:

- `allocation_percent`
  - normalized across a child's parents
  - preserves the child's total owned value across multiple parents
- `weight_input_type` + `weight_value`
  - normalized across a parent's children
  - controls local rollup behavior under that parent

For a shared child:

1. calculate the child's owned progress once
2. split it across parents using `allocation_percent`
3. under each parent, apply the configured weighting method among siblings
4. aggregate upward

This guarantees that:

- a child's total propagated progress across all parents never exceeds its owned progress
- changing one parent's local weighting does not silently change what another parent receives
- shared-child reuse remains mathematically stable and auditable

---

### 11.2 Budget Aggregation

```
FUNCTION aggregateBudget(elementId, strategyId):

  childLinks = QUERY direct outgoing edges for elementId in strategyId
               joined with child workflow status and budget fields

  includedLinks = childLinks WHERE child.status NOT IN ('DEACTIVATED', 'DEPRECATED')

  ownBudget = owned_budget_for(elementId)
  plannedTotal   = ownBudget.planned
  allocatedTotal = ownBudget.allocated
  consumedTotal  = ownBudget.consumed

  FOR EACH link IN includedLinks:
    childTotals = aggregateBudget(link.child_element_id, strategyId)
    childParentCount = COUNT_ACTIVE_PARENTS(link.child_element_id, strategyId)
    allocation = link.allocation_percent

    IF childParentCount <= 1 AND allocation IS NULL:
      allocation = 1.0

    plannedTotal   += childTotals.planned   × allocation
    allocatedTotal += childTotals.allocated × allocation
    consumedTotal  += childTotals.consumed  × allocation

  RETURN {
    planned:   plannedTotal,
    allocated: allocatedTotal,
    consumed:  consumedTotal,
    remaining: allocatedTotal - consumedTotal,
    variance:  plannedTotal - allocatedTotal
  }
```

**Budget propagation principle:**

- Sum all contribution percentages for each node and then divide contribution percent of each edge by that sum multiplied by budget so in that case allocated budget will be calculated not defined
- ~~A child budget belongs to the child once~~
- ~~When that child has multiple parents, its budget is split across those parents using incoming `allocation_percent`~~
- ~~Therefore a child with `200K` budget linked `60%` to Goal X and `40%` to Goal Y contributes `120K` to X and `80K` to Y, never `200K` to both~~

---

### 11.3 Element Health Score (Indicator-Based)

```
FUNCTION calculateHealthScore(elementId, strategyId):

  indicatorLinks = QUERY:
    SELECT iel.indicator_id, iel.weight_value, iel.weight_input_type,
           i.workflow_status
    FROM indicator_element_link iel
    JOIN indicator i ON i.id = iel.indicator_id
    WHERE iel.element_id = elementId
      AND iel.strategy_id = strategyId
      AND i.deleted_at IS NULL
      AND i.workflow_status = 'ACTIVE'

  IF indicatorLinks IS EMPTY:
    RETURN null  -- health score not computable

  -- Normalize weights using same logic as progress
  normalize each link's effectiveWeight

  totalWeight = SUM(link.effectiveWeight)

  fulfilledWeight = SUM(
    link.effectiveWeight
    FOR link IN indicatorLinks WHERE indicator(link.indicator_id).isFulfilled() = true
  )

  RETURN fulfilledWeight / totalWeight  -- 0.0 to 1.0, displayed as percentage

-- isFulfilled:
IF indicator.bigger_is_better:
  isFulfilled = (latestActualForCurrentPeriod >= currentPeriodTarget)
ELSE:
  isFulfilled = (latestActualForCurrentPeriod <= currentPeriodTarget)
```

---

### 11.4 Risk Score

```
FUNCTION calculateRiskScore(elementId, strategyId):

  riskLinks = QUERY:
    SELECT r.urgency_level
    FROM risk_element_link rel
    JOIN risk r ON r.id = rel.risk_id
    WHERE rel.element_id = elementId
      AND rel.strategy_id = strategyId
      AND r.deleted_at IS NULL
      AND r.workflow_status NOT IN ('RESOLVED', 'CLOSED', 'ACCEPTED')

  IF riskLinks IS EMPTY:
    RETURN { level: 'LOW', score: 0.0 }

  urgencyMap = { 'LOW': 1, 'MEDIUM': 2, 'HIGH': 3, 'CRITICAL': 4 }
  scores = [urgencyMap[r.urgency_level] FOR r IN riskLinks]

  -- 70% weight on max risk, 30% on average: surfaces critical risks prominently
  compositeScore = (0.7 × MAX(scores) + 0.3 × AVG(scores)) / 4.0

  riskLevel = CRITICAL if compositeScore >= 0.75
              HIGH     if compositeScore >= 0.50
              MEDIUM   if compositeScore >= 0.25
              else LOW

  RETURN { level: riskLevel, score: compositeScore }
```

---

### 11.5 Composite Metrics Response

```json
{
  "elementId": "...",
  "strategyId": "...",
  "progress": {
    "ownProgress": 72.0,
    "calculatedProgress": 68.5,
    "discrepancy": true,
    "discrepancyThreshold": 5.0,
    "source": "CALCULATED",
    "lastUpdated": "2026-03-29T14:00:00Z"
  },
  "budget": {
    "planned": { "amount": 500000, "currency": "AED" },
    "allocated": { "amount": 480000, "currency": "AED" },
    "consumed": { "amount": 310000, "currency": "AED" },
    "remaining": { "amount": 170000, "currency": "AED" },
    "variance": { "amount": 20000, "currency": "AED" }
  },
  "health": {
    "score": 0.75,
    "percentage": 75,
    "indicatorCount": 4,
    "fulfilledCount": 3
  },
  "risk": {
    "level": "MEDIUM",
    "score": 0.38,
    "activeRiskCount": 2
  }
}
```

---

## 12. PERMISSIONS MODEL

### Overview

The SPM module uses **OpenFGA** for all authorization. Every permission check is resolved through OpenFGA at runtime. User roles assigned in the application (owner, editor, viewer, module admin, strategy owner) are modeled as OpenFGA tuples. The rules below describe the permissions in business terms; OpenFGA translates them to computed permissions at access time.

Authorization must now evaluate two separate concerns:

- **Global element authority**: who owns, edits, shares, or deletes the underlying element record
- **Strategy-scoped usage authority**: who may import, link, weight, configure, and view that element inside a specific strategy

### Role Definitions

| Role                        | Scope                                          | Description                                                                                              |
| --------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **module_admin**            | SPM module                                     | Full administrative access to the SPM module including type configuration and all administrative actions |
| **strategy_owner**          | Single strategy                                | Owns and manages a specific strategy                                                                     |
| **strategy_editor**         | Single strategy                                | Can edit strategy configuration and manage element links within it                                       |
| **strategy_viewer**         | Single strategy                                | Read-only access to strategy and its elements                                                            |
| **element_owner**           | Single element                                 | Owns and manages a specific element globally (across all strategies)                                     |
| **element_editor**          | Single element                                 | Can edit a specific element's data globally                                                              |
| **element_viewer**          | Single element                                 | Read-only access to a specific element                                                                   |
| **shared_element_consumer** | Single shared/imported element in one strategy | Can use an imported element in strategy context without gaining ownership of the source element          |

### Top-Down Permission Inheritance

Permissions on a parent element propagate downward to all its children within the scope of a strategy. This is modeled in OpenFGA using recursive relation resolution.

**Rule**: If a user holds strategy-scoped inherited edit permission on Element P, and Element P has a child Element C in Strategy S, then the user may inherit strategy-scoped permissions on Element C within that strategy context, and recursively on all descendants of C.

This means: assigning a user as editor of a Goal can automatically grant them strategy-scoped access to child elements under that Goal in the strategy, without requiring explicit per-element assignments.

**Inheritance scope**: Inheritance is computed through the strategy graph. If Element C is a child of P in Strategy S but not in Strategy S2, the inherited permission applies only in Strategy S's context. Global element permissions (direct assignment) apply everywhere.

**Multi-parent rule**: If an element has multiple parents in the same strategy, inherited permissions are the **union of all active parent paths**. Removing one parent path removes only the permissions contributed by that path if no alternative path still grants them.

**Imported-element rule**: Inheritance from strategy structure never upgrades a consumer of an imported element into a global owner/editor of that source element. Strategy inheritance grants strategy-scoped use rights only unless the source owner separately grants global element rights.

**<u>TODO :: Ask for permission on specific spm element type</u>**

**<u>Permission to approve strategy</u>**

 **Permission to activate spm element one by one or all at the same time**

### Permission Matrix

| Action                                                         | Who Holds This Permission                                                                                                                                             |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Browse element library** (name, type, status)                | Module users (any user with a role in the SPM module — module_admin, or any element/strategy role)                                                                    |
| **View element full detail**                                   | element_viewer, element_editor, element_owner — OR — inherited from strategy membership (strategy_viewer, strategy_editor, strategy_owner of any containing strategy) |
| **Create a new element**                                       | Any module user                                                                                                                                                       |
| **Edit element global fields**                                 | element_editor, element_owner                                                                                                                                         |
| **Change element status**                                      | Per transition table in Section 6.5                                                                                                                                   |
| **Add / remove team member from element**                      | element_owner (manage_members permission)                                                                                                                             |
| **Share element for import by others**                         | element_owner, module_admin                                                                                                                                           |
| **Import shared element into strategy**                        | strategy_editor, strategy_owner on target strategy **and** `element:import` or source share grant on the shared element                                               |
| **Link element to a strategy**                                 | strategy_editor, strategy_owner of target strategy — OR — element_owner of the element being linked                                                                   |
| **Unlink element from a strategy**                             | strategy_editor, strategy_owner of target strategy — OR — element_owner of the element being unlinked                                                                 |
| **Update edge weight**                                         | strategy_editor, strategy_owner                                                                                                                                       |
| **Override strategy-scoped type defaults for a selected type** | strategy_editor, strategy_owner                                                                                                                                       |
| **Configure project linkage for a strategy type**              | strategy_editor, strategy_owner                                                                                                                                       |
| **Soft-delete an element**                                     | element_owner                                                                                                                                                         |
| **Restore a soft-deleted element**                             | module_admin OR strategy_owner of any strategy that contained the element                                                                                             |
| **Define / modify element types**                              | module_admin                                                                                                                                                          |
| **Create / modify strategy templates**                         | module_admin OR delegated template manager                                                                                                                            |
| **Create a strategy**                                          | Any module user                                                                                                                                                       |
| **Edit strategy configuration (type levels, weight mode)**     | strategy_owner, strategy_editor                                                                                                                                       |
| **Transition strategy status**                                 | Per transition table in Section 6.5                                                                                                                                   |
| **Link KPI to element**                                        | strategy_editor or strategy_owner on the strategy, and strategy-scoped permission for that element context                                                            |
| **Link risk to element**                                       | strategy_editor or strategy_owner on the strategy, and strategy-scoped permission for that element context                                                            |
| **Edit imported element global fields**                        | Only source `element_editor` / `element_owner`, not the importing strategy team by default                                                                            |
| **View element metrics**                                       | Any user with view access to the element                                                                                                                              |

### Permission Scenarios To Cover

- **Owned element in own strategy**: owner can edit global fields and strategy links
- **Shared element not yet imported**: non-owners may discover/import only if the owner shared it
- **Imported element in my strategy**: strategy owner/editor can link, unlink, weight, and configure strategy-scoped behavior, but cannot edit source global fields unless separately granted
- **Shared element used by many strategies**: revoking share blocks future imports and can optionally trigger review of existing imports; existing imports remain readable unless policy says otherwise
- **Exported reporting artifact**: export permission applies to the generated output, not to ownership of the source elements

### Practical Example With Named Users

Assume:

- **Ahmed** owns element `Citizen Portal`
- **Fatma** owns element `National Data Platform`
- **Omar** owns strategy `Digital Government 2030`
- **Sara** is editor on strategy `Digital Government 2030`
- **Khalid** is viewer on strategy `Digital Government 2030`

#### Example 1: Owned Element In Own Strategy

Ahmed creates `Citizen Portal`, so Ahmed is the global `element_owner`.

If Ahmed also uses `Citizen Portal` in his own strategy:

- Ahmed may rename the element
- Ahmed may edit description and ownership/team
- Ahmed may link it under strategy parents
- Ahmed may update edge allocation and weighting in that strategy

This is valid because Ahmed holds both:

- global element authority on `Citizen Portal`
- strategy authority in the strategy where it is used

#### Example 2: Shared Element Before Import

Fatma owns `National Data Platform` and marks it as shared for import.

Omar, as owner of strategy `Digital Government 2030`, may:

- discover the element in the shared library
- view enough metadata to evaluate import
- import it into his strategy if the share grant permits

Before import, Omar may not edit the source element and may not use it in his strategy graph yet.

#### Example 3: Imported Element In My Strategy

Omar imports `National Data Platform` into `Digital Government 2030`.

Now inside Omar's strategy:

- Omar may link it under `Open Data Goal`
- Sara may adjust edge allocation and local edge weighting in this strategy
- Khalid may view it in this strategy

But:

- Omar may not rename `National Data Platform` globally
- Sara may not change its global description
- Khalid may not manage its share settings

Fatma remains the source `element_owner`, so global authority stays with Fatma while strategy-scoped usage authority belongs to Omar's strategy team.

#### Example 4: Strategy-Scoped Inheritance

Inside `Digital Government 2030`:

- Parent goal: `Open Data Goal`
- Child initiative: `National Data Platform`

Sara is assigned as editor on `Open Data Goal`.

Then Sara inherits strategy-scoped rights on `National Data Platform` within `Digital Government 2030`. This inherited access may allow Sara to:

- view the element in this strategy context
- manage strategy-context linkage and weighting if her strategy role allows
- operate on descendants reachable through that path in the same strategy

But Sara still may not:

- rename the element globally
- transfer ownership
- edit source-global fields unless Fatma separately grants global element rights

#### Example 5: Multi-Parent Inheritance

`National Data Platform` is linked under two goals in the same strategy:

- `Open Data Goal`
- `AI Enablement Goal`

Sara is editor on `Open Data Goal`.
Mariam is editor on `AI Enablement Goal`.

Then both Sara and Mariam may gain strategy-scoped rights on `National Data Platform` in `Digital Government 2030`.

This is because inherited permissions are computed as the **union of active parent paths**:

- Sara receives inherited access through `Open Data Goal`
- Mariam receives inherited access through `AI Enablement Goal`

If the `Open Data Goal -> National Data Platform` link is removed:

- Sara loses inherited access from that path unless another active path still grants it
- Mariam keeps inherited access through `AI Enablement Goal`

#### Example 6: Import Does Not Transfer Ownership

Fatma shares `National Data Platform`.
Omar imports it.
Sara edits its position and link settings in Omar's strategy.

Still:

- Fatma remains the source owner
- only Fatma, or users Fatma granted as `element_editor`, may change:
  - name
  - core description
  - ownership
  - share settings
  - delete/restore

Importing an element grants strategy usage rights only. It does not transfer source ownership.

#### Example 7: Revoking Share

Fatma later revokes sharing on `National Data Platform`.

Then:

- no new strategy may import it
- existing imports may remain readable and usable depending on policy
- existing importing teams still do not become owners

Recommended default policy:

- revoking share blocks future imports
- already imported usages remain readable in already-linked strategies
- source ownership and global edit rights remain with Fatma's source-side team

### Why These Examples Are Compatible With OpenFGA

The examples above are directly compatible with an OpenFGA model because they separate:

- **direct global relations** on the element object
  - `owner`
  - `editor`
  - `viewer`
- **direct strategy relations** on the strategy object
  - `owner`
  - `editor`
  - `viewer`
- **share/import relations**
  - source element shared with a target strategy or user
- **graph inheritance relations**
  - parent-child links resolved in strategy context

In OpenFGA terms:

- Fatma is directly related to `National Data Platform` as `owner`
- Omar and Sara are directly related to `Digital Government 2030` as `owner` / `editor`
- the imported usage is granted by share/import relation plus strategy membership
- inherited strategy-scoped access is computed through parent-child graph relations within the strategy context
- no inherited or imported relation writes Omar or Sara into the element's global `owner` or `editor` relation unless Fatma explicitly grants it

This preserves the required business rule:

- imported/shared usage may expand **strategy-scoped** rights
- only direct source-side grants may expand **global element** rights

### OpenFGA Model Summary

```
type spm_element {
  relations {
    owner:  [user]
    editor: [user]
    viewer: [user]
    parent: [spm_element]          // for top-down inheritance
    shared_with: [user, strategy]
  }
  permissions {
    can_view   = viewer or editor or owner
                 or parent.can_view         // top-down
                 or strategy_viewer (any containing strategy)
    can_edit   = editor or owner
    can_delete = owner
    can_manage_members = owner
    can_share  = owner or module_admin
    can_import = shared_with or owner or editor
  }
}

type strategy {
  relations {
    owner:  [user]
    editor: [user]
    viewer: [user]
  }
  permissions {
    can_view         = viewer or editor or owner
    can_edit         = editor or owner
    can_link_element = editor or owner
    can_configure_effective_types = editor or owner
    can_import_shared_element = editor or owner
  }
}

type spm_module {
  relations {
    admin: [user]
  }
  permissions {
    can_administer = admin
    can_use        = admin or [any strategy/element role holder]
  }
}
```

---

## 13. NAVIGATION MODEL

### Default Navigation

The default navigation of a strategy follows its type level configuration (e.g., Perspective → Goal → Initiative). This is the standard strategy map view.

### Alternative Navigation Views

Navigation views are derived automatically from the graph — not stored. The system generates views based on the element types present in the strategy.

Available auto-generated views:

- **Default Map**: top-down following type level order
- **By [Type Name]**: flat list of all elements of a given type (e.g., "All Goals")
- **[Type A] → [Type B]**: two-level collapsed map skipping intermediate types
- **Full Graph**: all nodes and edges as an interactive network

### User Navigation Preference

Each user's **preferred view per strategy** is persisted in `user_strategy_nav_preference`. This is a user-level setting. When a user opens a strategy, they see their last chosen view. No strategy-level default view is stored — only per-user preferences.

---

## 14. TECHNICAL DATA MODEL

### Core Tables

#### `spm_element_type`

```sql
CREATE TABLE spm_element_type (
    id               BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    public_id        UUID NOT NULL DEFAULT gen_random_uuid(),
    organization_id  UUID NOT NULL,
    code             VARCHAR(50) NOT NULL,
    name             JSONB NOT NULL,
    description      JSONB,
    icon             VARCHAR(100),
    color            VARCHAR(20),
    is_budgetable    BOOLEAN NOT NULL DEFAULT false,
    is_trackable     BOOLEAN NOT NULL DEFAULT false,
    has_kpis         BOOLEAN NOT NULL DEFAULT false,
    has_risks        BOOLEAN NOT NULL DEFAULT false,
    linkable_with_project BOOLEAN NOT NULL DEFAULT false,
    is_vision_aligned BOOLEAN NOT NULL DEFAULT false,
    is_quarterly_measurable BOOLEAN NOT NULL DEFAULT false,
    display_order    INTEGER NOT NULL DEFAULT 0,
    created_at       TIMESTAMP WITHOUT TIME ZONE,
    created_by       VARCHAR(255),
    last_modified_at TIMESTAMP WITHOUT TIME ZONE,
    last_modified_by VARCHAR(255),
    deleted_at       TIMESTAMP WITHOUT TIME ZONE,
    deleted_by       VARCHAR(255),
    CONSTRAINT uq_element_type_org_code UNIQUE (organization_id, code)
);
```

#### `spm_element`

```sql
CREATE TABLE spm_element (
    id                   BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    public_id            UUID NOT NULL DEFAULT gen_random_uuid(),
    organization_id      UUID NOT NULL,
    type_id              BIGINT NOT NULL REFERENCES spm_element_type(id),
    name                 JSONB NOT NULL,
    description          JSONB,
    start_date           DATE,
    end_date             DATE,
    workflow_status      VARCHAR(20) NOT NULL DEFAULT 'DRAFT',
    organization_unit_id UUID,
    version              BIGINT NOT NULL DEFAULT 0,
    created_at           TIMESTAMP WITHOUT TIME ZONE,
    created_by           VARCHAR(255),
    last_modified_at     TIMESTAMP WITHOUT TIME ZONE,
    last_modified_by     VARCHAR(255),
    deleted_at           TIMESTAMP WITHOUT TIME ZONE,
    deleted_by           VARCHAR(255),
    CONSTRAINT chk_element_status CHECK (
        workflow_status IN ('DRAFT','ACTIVE','ON_HOLD','COMPLETED','DEACTIVATED','DEPRECATED')
    ),
    CONSTRAINT chk_element_date_order CHECK (
        end_date IS NULL OR end_date >= start_date
    )
);

CREATE INDEX idx_spm_element_org    ON spm_element(organization_id);
CREATE INDEX idx_spm_element_type   ON spm_element(type_id);
CREATE INDEX idx_spm_element_status ON spm_element(workflow_status);
CREATE INDEX idx_spm_element_org_status ON spm_element(organization_id, workflow_status);
```

#### `spm_element_status_history`

```sql
CREATE TABLE spm_element_status_history (
    id           BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    element_id   BIGINT NOT NULL REFERENCES spm_element(id),
    from_status  VARCHAR(20),
    to_status    VARCHAR(20) NOT NULL,
    changed_at   TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    changed_by   VARCHAR(255) NOT NULL,
    note         TEXT
);

CREATE INDEX idx_status_history_element ON spm_element_status_history(element_id);
```

#### `spm_element_team_member`

```sql
CREATE TABLE spm_element_team_member (
    id          BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    element_id  BIGINT NOT NULL REFERENCES spm_element(id) ON DELETE CASCADE,
    user_id     UUID NOT NULL,
    role        VARCHAR(20) NOT NULL,  -- OWNER, EDITOR, VIEWER
    added_at    TIMESTAMP WITHOUT TIME ZONE,
    added_by    VARCHAR(255),
    CONSTRAINT uq_element_team_member UNIQUE (element_id, user_id),
    CONSTRAINT chk_member_role CHECK (role IN ('OWNER','EDITOR','VIEWER'))
);
```

### Capability Extension Tables

#### `spm_element_budget` *(Budgetable capability — includes payment tracking)*

```sql
CREATE TABLE spm_element_budget (
    element_id       BIGINT PRIMARY KEY REFERENCES spm_element(id) ON DELETE CASCADE,
    planned_budget   JSONB,
    allocated_budget JSONB
);
```

#### `spm_element_budget_source`

```sql
CREATE TABLE spm_element_budget_source (
    id          BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    element_id  BIGINT NOT NULL REFERENCES spm_element(id) ON DELETE CASCADE,
    source_type VARCHAR(50) NOT NULL
);
```

#### `spm_element_payment` *(part of Budgetable capability)*

```sql
CREATE TABLE spm_element_payment (
    id               BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    public_id        UUID NOT NULL DEFAULT gen_random_uuid(),
    element_id       BIGINT NOT NULL REFERENCES spm_element(id),
    title            VARCHAR(255),
    description      TEXT,
    payment_date     DATE NOT NULL,
    payment_time     TIME,
    amount           JSONB NOT NULL,
    created_at       TIMESTAMP WITHOUT TIME ZONE,
    created_by       VARCHAR(255),
    last_modified_at TIMESTAMP WITHOUT TIME ZONE,
    last_modified_by VARCHAR(255),
    deleted_at       TIMESTAMP WITHOUT TIME ZONE,
    deleted_by       VARCHAR(255)
);

CREATE INDEX idx_payment_element ON spm_element_payment(element_id);
```

#### `spm_element_progress` *(Trackable capability)*

```sql
CREATE TABLE spm_element_progress (
    id           BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    element_id   BIGINT NOT NULL REFERENCES spm_element(id) ON DELETE CASCADE,
    entry_date   TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    rate         DOUBLE PRECISION NOT NULL,
    entry_type   VARCHAR(20) NOT NULL,
    note         TEXT,
    created_by   VARCHAR(255),
    CONSTRAINT chk_progress_rate  CHECK (rate >= 0 AND rate <= 100),
    CONSTRAINT chk_progress_type  CHECK (entry_type IN ('MILESTONE', 'PROGRESS_LOG'))
);

CREATE INDEX idx_progress_element      ON spm_element_progress(element_id);
CREATE INDEX idx_progress_element_date ON spm_element_progress(element_id, entry_date DESC);
```

#### `spm_element_calculated_progress` *(stores last calculated_progress per element per strategy)*

```sql
CREATE TABLE spm_element_calculated_progress (
    element_id          BIGINT NOT NULL REFERENCES spm_element(id),
    strategy_id         BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    calculated_progress DOUBLE PRECISION,
    discrepancy_flag    BOOLEAN NOT NULL DEFAULT false,
    last_computed_at    TIMESTAMP WITHOUT TIME ZONE,
    PRIMARY KEY (element_id, strategy_id)
);
```

#### `spm_element_vision_alignment` *(VisionAligned capability)*

```sql
CREATE TABLE spm_element_vision_alignment (
    element_id         BIGINT PRIMARY KEY REFERENCES spm_element(id) ON DELETE CASCADE,
    vision_priority_id BIGINT REFERENCES vision_priority(id)
);

CREATE TABLE spm_element_vision_goal (
    element_id     BIGINT NOT NULL REFERENCES spm_element(id) ON DELETE CASCADE,
    vision_goal_id BIGINT NOT NULL REFERENCES vision_goal(id),
    PRIMARY KEY (element_id, vision_goal_id)
);

CREATE TABLE spm_element_vision_indicator (
    element_id          BIGINT NOT NULL REFERENCES spm_element(id) ON DELETE CASCADE,
    vision_indicator_id BIGINT NOT NULL REFERENCES vision_indicator(id),
    PRIMARY KEY (element_id, vision_indicator_id)
);
```

### Strategy Tables

#### `strategy` *(existing table — no structural changes)*

```sql
-- Existing table kept as-is.
-- workflow_status values for strategy: DRAFT, ACTIVE, SUSPENDED, COMPLETED, ARCHIVED
-- planned_budget and allocated_budget remain as strategy-level own budget fields.
```

#### `strategy_type_level`

```sql
CREATE TABLE strategy_type_level (
    id                      BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    strategy_id             BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    element_type_id         BIGINT NOT NULL REFERENCES spm_element_type(id),
    level_order             INTEGER NOT NULL,
    is_budgetable           BOOLEAN NOT NULL DEFAULT false,
    is_trackable            BOOLEAN NOT NULL DEFAULT false,
    has_kpis                BOOLEAN NOT NULL DEFAULT false,
    has_risks               BOOLEAN NOT NULL DEFAULT false,
    linkable_with_project   BOOLEAN NOT NULL DEFAULT false,
    is_vision_aligned       BOOLEAN NOT NULL DEFAULT false,
    is_quarterly_measurable BOOLEAN NOT NULL DEFAULT false,
    display_config          JSONB,
    CONSTRAINT uq_strategy_type_level  UNIQUE (strategy_id, element_type_id),
    CONSTRAINT uq_strategy_level_order UNIQUE (strategy_id, level_order)
);
```

#### `strategy_config` *(strategy-level defaults)*

```sql
CREATE TABLE strategy_config (
    strategy_id              BIGINT PRIMARY KEY REFERENCES strategy(id) ON DELETE CASCADE,
    default_weight_input_type VARCHAR(10) NOT NULL DEFAULT 'IMPACT',
    discrepancy_threshold     DOUBLE PRECISION NOT NULL DEFAULT 5.0,
    CONSTRAINT chk_weight_input CHECK (default_weight_input_type IN ('PERCENT','IMPACT','EQUAL','BUSINESS_VALUE','BUDGETABLE'))
);
```

#### `strategy_template`

```sql
CREATE TABLE strategy_template (
    id                      BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    public_id               UUID NOT NULL DEFAULT gen_random_uuid(),
    organization_id         UUID NOT NULL,
    name                    VARCHAR(255) NOT NULL,
    description             TEXT,
    category                VARCHAR(100),
    version_label           VARCHAR(50),
    default_weight_input_type VARCHAR(10) NOT NULL DEFAULT 'IMPACT',
    discrepancy_threshold   DOUBLE PRECISION NOT NULL DEFAULT 5.0,
    strict_hierarchy        BOOLEAN NOT NULL DEFAULT true,
    created_at              TIMESTAMP WITHOUT TIME ZONE,
    created_by              VARCHAR(255),
    last_modified_at        TIMESTAMP WITHOUT TIME ZONE,
    last_modified_by        VARCHAR(255),
    deleted_at              TIMESTAMP WITHOUT TIME ZONE,
    deleted_by              VARCHAR(255)
);
```

#### `strategy_template_type_level`

```sql
CREATE TABLE strategy_template_type_level (
    id                      BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    template_id             BIGINT NOT NULL REFERENCES strategy_template(id) ON DELETE CASCADE,
    element_type_id         BIGINT NOT NULL REFERENCES spm_element_type(id),
    level_order             INTEGER NOT NULL,
    is_budgetable           BOOLEAN NOT NULL DEFAULT false,
    is_trackable            BOOLEAN NOT NULL DEFAULT false,
    has_kpis                BOOLEAN NOT NULL DEFAULT false,
    has_risks               BOOLEAN NOT NULL DEFAULT false,
    linkable_with_project   BOOLEAN NOT NULL DEFAULT false,
    is_vision_aligned       BOOLEAN NOT NULL DEFAULT false,
    is_quarterly_measurable BOOLEAN NOT NULL DEFAULT false,
    can_be_root             BOOLEAN NOT NULL DEFAULT false,
    display_config          JSONB,
    CONSTRAINT uq_template_type UNIQUE (template_id, element_type_id),
    CONSTRAINT uq_template_level UNIQUE (template_id, level_order)
);
```

#### `strategy_element` *(explicit membership — required for root nodes)*

```sql
CREATE TABLE strategy_element (
    strategy_id BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    element_id  BIGINT NOT NULL REFERENCES spm_element(id),
    added_at    TIMESTAMP WITHOUT TIME ZONE,
    added_by    VARCHAR(255),
    PRIMARY KEY (strategy_id, element_id)
);
```

#### `user_strategy_nav_preference`

```sql
CREATE TABLE user_strategy_nav_preference (
    user_id     UUID NOT NULL,
    strategy_id BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    view_type   VARCHAR(50) NOT NULL,
    updated_at  TIMESTAMP WITHOUT TIME ZONE,
    PRIMARY KEY (user_id, strategy_id)
);
```

### Graph Edge Table

#### `spm_element_link`

```sql
CREATE TABLE spm_element_link (
    id                  BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    public_id           UUID NOT NULL DEFAULT gen_random_uuid(),
    strategy_id         BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    parent_element_id   BIGINT NOT NULL REFERENCES spm_element(id),
    child_element_id    BIGINT NOT NULL REFERENCES spm_element(id),
    allocation_percent  DECIMAL(7,4),
    weight_value        DECIMAL(12,4),
    weight_input_type   VARCHAR(10) NOT NULL DEFAULT 'EQUAL',
    display_order       INTEGER NOT NULL DEFAULT 0,
    created_at          TIMESTAMP WITHOUT TIME ZONE,
    created_by          VARCHAR(255),
    deleted_at          TIMESTAMP WITHOUT TIME ZONE,
    deleted_by          VARCHAR(255),
    CONSTRAINT uq_strategy_edge      UNIQUE (strategy_id, parent_element_id, child_element_id),
    CONSTRAINT chk_no_self_link      CHECK  (parent_element_id <> child_element_id),
    CONSTRAINT chk_allocation_range  CHECK  (allocation_percent IS NULL OR (allocation_percent >= 0 AND allocation_percent <= 1.0000)),
    CONSTRAINT chk_weight_value_nonneg CHECK (weight_value IS NULL OR weight_value >= 0),
    CONSTRAINT chk_weight_input_type CHECK  (weight_input_type IN ('PERCENT','IMPACT','EQUAL','BUSINESS_VALUE','BUDGETABLE'))
);

CREATE INDEX idx_link_strategy ON spm_element_link(strategy_id);
CREATE INDEX idx_link_parent   ON spm_element_link(parent_element_id);
CREATE INDEX idx_link_child    ON spm_element_link(child_element_id);
```

### Indicator & Risk Link Tables

#### `indicator_element_link`

```sql
CREATE TABLE indicator_element_link (
    id                BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    indicator_id      BIGINT NOT NULL REFERENCES indicator(id),
    element_id        BIGINT NOT NULL REFERENCES spm_element(id),
    strategy_id       BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    weight_value      DECIMAL(12,4),
    weight_input_type VARCHAR(10) NOT NULL DEFAULT 'EQUAL',
    created_at        TIMESTAMP WITHOUT TIME ZONE,
    created_by        VARCHAR(255),
    CONSTRAINT uq_indicator_element_link UNIQUE (indicator_id, element_id, strategy_id),
    CONSTRAINT chk_ind_weight_value_nonneg CHECK (weight_value IS NULL OR weight_value >= 0),
    CONSTRAINT chk_ind_weight_type CHECK (weight_input_type IN ('PERCENT','IMPACT','EQUAL','BUSINESS_VALUE','BUDGETABLE'))
);
```

#### `risk_element_link`

```sql
CREATE TABLE risk_element_link (
    id          BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    risk_id     BIGINT NOT NULL REFERENCES risk(id),
    element_id  BIGINT NOT NULL REFERENCES spm_element(id),
    strategy_id BIGINT NOT NULL REFERENCES strategy(id) ON DELETE CASCADE,
    created_at  TIMESTAMP WITHOUT TIME ZONE,
    created_by  VARCHAR(255),
    CONSTRAINT uq_risk_element_link UNIQUE (risk_id, element_id, strategy_id)
);
```

### Specialized Entity Tables (Unchanged — Reference)

`indicator` — Retains all existing fields and tables (baseline, target, actual, team members).
`risk` — Retains all existing fields and tables (phases, impact assessment, mitigation plan, team members).

### Old Tables (To Be Migrated Then Dropped)

| Old Table                   | Migration Target                           | Notes                                                   |
| --------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| `perspective`               | `spm_element` (type=PERSPECTIVE)           |                                                         |
| `goal`                      | `spm_element` (type=GOAL)                  |                                                         |
| `initiative`                | `spm_element` (type=INITIATIVE)            |                                                         |
| `perspective.parent_id`     | `spm_element_link` edge                    |                                                         |
| `goal.parent_id`            | `spm_element_link` edge                    |                                                         |
| `initiative.parent_id`      | `spm_element_link` edge                    |                                                         |
| `initiative_impact_on_goal` | `spm_element_link`                         | impact_score → weight_value, weight_input_type = IMPACT |
| `goal_indicators`           | `indicator_element_link`                   |                                                         |
| `initiative_indicators`     | `indicator_element_link`                   |                                                         |
| `goal_risks`                | `risk_element_link`                        |                                                         |
| `initiative_risks`          | `risk_element_link`                        |                                                         |
| `initiative_milestones`     | `spm_element_progress` (type=MILESTONE)    |                                                         |
| `initiative_progress_log`   | `spm_element_progress` (type=PROGRESS_LOG) |                                                         |
| `initiative_budget_sources` | `spm_element_budget_source`                |                                                         |
| `payment`                   | `spm_element_payment`                      |                                                         |

---

## 15. API DESIGN

### Element Type Management

```
GET    /organizations/{orgId}/element-types           List all element types (module users only)
POST   /organizations/{orgId}/element-types           Create element type (module admin only)
GET    /organizations/{orgId}/element-types/{id}      Get element type detail
PUT    /organizations/{orgId}/element-types/{id}      Update element type (module admin only)
DELETE /organizations/{orgId}/element-types/{id}      Soft-delete (module admin, blocked if elements exist)

GET    /organizations/{orgId}/strategy-templates      List templates
POST   /organizations/{orgId}/strategy-templates      Create template
GET    /organizations/{orgId}/strategy-templates/{id} Get template detail
PUT    /organizations/{orgId}/strategy-templates/{id} Update template
DELETE /organizations/{orgId}/strategy-templates/{id} Soft-delete template
```

### Element Library

```
GET    /elements                                      Search/list elements (module users only)
                                                      Params: typeId, status, orgUnit, search, page, size, includeDeprecated
POST   /elements                                      Create element (module users only)
GET    /elements/{id}                                 Get element detail
PUT    /elements/{id}                                 Update global fields
DELETE /elements/{id}                                 Soft-delete

POST   /elements/{id}/share                           Share element for import { targetUsers?, targetStrategies?, accessPolicy }
DELETE /elements/{id}/share/{shareId}                 Revoke share
GET    /elements/{id}/imports                         List strategies where this shared element is imported

PATCH  /elements/{id}/status                          Transition status { toStatus, note }
GET    /elements/{id}/status-history                  Status audit trail

-- Budgetable sub-resources
GET    /elements/{id}/budget                          Get budget data
PUT    /elements/{id}/budget                          Set/update budget
GET    /elements/{id}/payments                        List payments (paginated)
POST   /elements/{id}/payments                        Create payment
PUT    /elements/{id}/payments/{pid}                  Update payment
DELETE /elements/{id}/payments/{pid}                  Soft-delete payment

-- Trackable sub-resources
GET    /elements/{id}/progress                        List all progress entries + milestones
POST   /elements/{id}/progress                        Add progress log entry
POST   /elements/{id}/milestones                      Add milestone
PUT    /elements/{id}/progress/{eid}                  Update entry
DELETE /elements/{id}/progress/{eid}                  Delete entry
GET    /elements/{id}/calculated-progress?strategyId  Get calculated_progress + discrepancy for strategy context

-- VisionAligned sub-resources
GET    /elements/{id}/vision-alignment                Get vision alignment
PUT    /elements/{id}/vision-alignment                Set/update vision alignment

-- Team members
GET    /elements/{id}/team-members                    List team members
POST   /elements/{id}/team-members                    Add team member { userId, role }
PUT    /elements/{id}/team-members/{uid}              Update member role
DELETE /elements/{id}/team-members/{uid}              Remove team member

-- Context queries
GET    /elements/{id}/strategies                      All strategies this element belongs to
GET    /elements/{id}/metrics?strategyId={sid}        Composite metrics (progress + budget + health + risk)
```

### Strategy Graph Operations

```
-- Build configuration
GET    /strategies/{id}/config                        Get strategy config (weight input type, discrepancy threshold)
PUT    /strategies/{id}/config                        Update strategy config
GET    /strategies/{id}/type-levels                   Get type level configuration
PUT    /strategies/{id}/type-levels                   Set type level configuration

-- Strategy status
PATCH  /strategies/{id}/status                        Transition strategy status { toStatus, note }
GET    /strategies/{id}/status-history                Strategy status audit trail

-- Element membership
GET    /strategies/{id}/elements                      All elements in strategy
POST   /strategies/{id}/elements                      Add element as root node { elementId }
DELETE /strategies/{id}/elements/{elementId}          Remove element (removes all its edges in strategy)

-- Edge operations
GET    /strategies/{id}/links                         All edges in strategy
POST   /strategies/{id}/links                         Create edge { parentElementId, childElementId, allocationPercent?, weightValue?, weightInputType?, displayOrder? }
PUT    /strategies/{id}/links/{linkId}                Update allocation, weighting method, or display order
DELETE /strategies/{id}/links/{linkId}                Remove edge

-- Graph queries
GET    /strategies/{id}/elements/{eid}/children       Direct children
GET    /strategies/{id}/elements/{eid}/parents        Direct parents (zero to many)
GET    /strategies/{id}/elements/{eid}/descendants    All descendants (DFS)
GET    /strategies/{id}/graph                         Full graph payload { nodes[], edges[], typeConfig[], strategyConfig }

-- Aggregate metrics
GET    /strategies/{id}/metrics                       Top-level aggregated progress + budget for entire strategy

-- Indicator links in strategy
GET    /strategies/{id}/indicator-links               List all indicator links
POST   /strategies/{id}/indicator-links               { indicatorId, elementId, weightValue?, weightInputType? }
PUT    /strategies/{id}/indicator-links/{lid}         Update weight
DELETE /strategies/{id}/indicator-links/{lid}         Remove link

-- Risk links in strategy
GET    /strategies/{id}/risk-links                    List all risk links
POST   /strategies/{id}/risk-links                    { riskId, elementId }
DELETE /strategies/{id}/risk-links/{lid}              Remove link
```

### Navigation Preference

```
GET  /users/me/nav-preferences/{strategyId}           Get user's preferred view for strategy
PUT  /users/me/nav-preferences/{strategyId}           Set preferred view { viewType }
```

---

## 16. FRONTEND ARCHITECTURE

### Key Structural Changes

**What changes:**

- Strategy detail view fetches full graph data and renders it dynamically — no hardcoded type traversal
- All entity-specific forms replaced by a single **ElementForm** that conditionally renders sections based on type capabilities
- All element pickers open the **Global Element Library** panel (accessible to module users only)
- Parent selectors replaced by edge management
- Strategy creation supports **From Template** and **From Scratch**
- Strategy type configuration stores effective per-strategy capability overrides copied from type defaults
- Status transitions handled via a dedicated **StatusFlowWidget**
- Progress display shows both `own_progress` and `calculated_progress` with discrepancy indicator where applicable

**What stays the same:**

- React Query (TanStack) for server state
- Context API for strategy labels and permissions
- Zod schemas for validation
- Localized field inputs (en/ar)
- Team member management patterns

### New Components Required

| Component                      | Purpose                                                                                                                                          |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ElementForm`                  | Generic form. Sections rendered by capability. Fields locked by status per Section 6.4.                                                          |
| `ElementLibraryPanel`          | Slide-over for discovering and linking elements. Accessible to module users only.                                                                |
| `ElementTypeBadge`             | Type name with configured color/icon.                                                                                                            |
| `CrossStrategyBadge`           | Shows "Used in X strategies" on elements shared across strategies.                                                                               |
| `StatusFlowWidget`             | Current status display, allowed transition buttons, note input for transitions.                                                                  |
| `StatusHistoryTimeline`        | Timeline of all status changes with actor, timestamp, and optional note.                                                                         |
| `ProgressDisplay`              | Shows both `own_progress` and `calculated_progress`. Displays discrepancy indicator when both exist and differ.                                  |
| `GraphStrategyMap`             | ECharts/D3 network visualization. Nodes typed by color/icon. Status reflected in node opacity. Edge weight shown on hover.                       |
| `WeightInput`                  | Supports all configured edge weighting methods and shows effective normalized weight. Also captures parent allocation share for shared children. |
| `ElementTypeConfigurator`      | Module admin screen: define/edit types, set capabilities, icon, color, order.                                                                    |
| `StrategyTemplatePicker`       | Create strategy flow: choose template or start from scratch.                                                                                     |
| `StrategyTemplateConfigurator` | Create/edit reusable templates with type rules and default overrides.                                                                            |
| `StrategyTypeLevelConfig`      | Strategy settings: type levels, default weight input type, discrepancy threshold.                                                                |
| `NavViewSelector`              | Switch between Default Map / By Type / Full Graph views.                                                                                         |
| `CapabilitySection`            | Reusable form section (budget, progress, vision). Checks capability flag before rendering.                                                       |

### Modified Hooks

| Hook                         | Change                                                                                         |
| ---------------------------- | ---------------------------------------------------------------------------------------------- |
| `useStrategyResources`       | Complete rewrite — fetches `/strategies/{id}/graph`, builds graph structure client-side        |
| `useCreateElement`           | Replaces useCreateGoal, useCreatePerspective, useCreateInitiative                              |
| `useUpdateElement`           | Generic update                                                                                 |
| `useTransitionElementStatus` | PATCH /elements/{id}/status — confirmation dialog for irreversible transitions                 |
| `useElementStatusHistory`    | GET /elements/{id}/status-history                                                              |
| `useCreateLink`              | POST /strategies/{id}/links — includes client-side cycle check and hierarchy conformance check |
| `useDeleteLink`              | DELETE /strategies/{id}/links/{id}                                                             |
| `useUpdateLinkWeight`        | PUT /strategies/{id}/links/{id}                                                                |
| `useElementMetrics`          | GET /elements/{id}/metrics?strategyId — composite metrics                                      |
| `useElementLibrary`          | GET /elements with debounced search, type/status filters. Available to module users only.      |
| `useStrategyGraph`           | GET /strategies/{id}/graph                                                                     |
| `useNavPreference`           | GET + PUT /users/me/nav-preferences/{strategyId}                                               |
| `useStrategyConfig`          | GET + PUT /strategies/{id}/config                                                              |

### Status-Based Field Locking

`ElementForm` enforces locking based on element status:

| Field Group        | DRAFT       | ACTIVE           | ON_HOLD          | COMPLETED | DEACTIVATED | DEPRECATED |
| ------------------ | ----------- | ---------------- | ---------------- | --------- | ----------- | ---------- |
| Name, Type         | ✏️ editable | ✏️ (type locked) | 🔒               | 🔒        | 🔒          | 🔒         |
| Description        | ✏️          | ✏️               | ✏️               | 🔒        | 🔒          | 🔒         |
| Start date         | ✏️          | 🔒               | 🔒               | 🔒        | 🔒          | 🔒         |
| End date           | ✏️          | ✏️               | ✏️ (extend only) | 🔒        | 🔒          | 🔒         |
| Org unit           | ✏️          | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |
| Budget (planned)   | ✏️          | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |
| Budget (allocated) | ✏️          | ✏️               | ✏️               | 🔒        | 🔒          | 🔒         |
| Progress entry     | ➖ n/a       | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |
| Indicator links    | ✏️          | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |
| Risk links         | ✏️          | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |
| Vision alignment   | ✏️          | ✏️               | 🔒               | 🔒        | 🔒          | 🔒         |

### Real-Time Updates (WebSocket/SSE)

When a screen is displaying a strategy map or summary that includes elements whose calculations may be affected by an ongoing status change or progress update:

- Subscribe to a strategy-scoped notification channel via WebSocket/SSE
- On receiving a recalculation event for an element in the graph, invalidate the affected React Query cache keys for that element's metrics and all ancestor metrics
- React Query refetches automatically, updating displayed values without full page reload

---

## 17. MIGRATION STRATEGY

### Phase 1 — Additive (No Breaking Changes)

1. Create all new tables (all tables defined in Section 14)
2. Implement and expose new API endpoints alongside existing ones
3. Old API endpoints continue to function — no changes to existing behavior
4. Deploy but do not activate new frontend features

### Phase 2 — Data Migration

Execute in sequence, within a transaction per organization:

1. Create default element types for each organization (PERSPECTIVE, GOAL, INITIATIVE) with default strategy-start capabilities populated, including KPI/risk/project-link flags where applicable
2. Migrate `perspective` → `spm_element` (type=PERSPECTIVE). Populate `spm_element_team_member` from existing perspective team members
3. Migrate `goal` → `spm_element` (type=GOAL). Migrate goal budget → `spm_element_budget`. Migrate goal team members.
4. Migrate `initiative` → `spm_element` (type=INITIATIVE). Migrate initiative budget → `spm_element_budget`. Migrate budget sources → `spm_element_budget_source`. Migrate team members.
5. Migrate all `payment` records → `spm_element_payment`
6. Migrate `initiative_milestones` and `initiative_progress_log` → `spm_element_progress`
7. Migrate `perspective.parent_id` → `spm_element_link` edges
8. Migrate `goal.parent_id` → `spm_element_link` edges
9. Migrate `initiative.parent_id` → `spm_element_link` edges
10. Migrate `initiative_impact_on_goal` → `spm_element_link` (impact_score → `weight_value`, `weight_input_type = IMPACT`)
11. Migrate `goal_indicators` and `initiative_indicators` → `indicator_element_link`
12. Migrate `goal_risks` and `initiative_risks` → `risk_element_link`
13. Populate `strategy_element` for all root-level elements
14. Create initial `spm_element_status_history` record for every migrated element
15. Create `strategy_config` rows with default weight input type and discrepancy threshold for all strategies
16. Backfill `strategy_type_level` effective capability values from the source `spm_element_type` defaults for every existing strategy/type pairing
17. Seed baseline strategy templates for organizations that use common repeatable strategy models
18. Compute and store initial `spm_element_calculated_progress` for all active elements using DAG-safe aggregation
19. Verify row counts match between old and new tables before proceeding to Phase 3

### Phase 3 — Switch Frontend

1. Regenerate API client types from updated OpenAPI spec
2. Replace strategy detail view with graph-aware implementation
3. Replace all entity-specific forms with generic `ElementForm`
4. Implement `ElementLibraryPanel` and wire all link dialogs to it
5. Implement `StatusFlowWidget` and `StatusHistoryTimeline`
6. Implement `ProgressDisplay` with discrepancy indicator
7. Implement `GraphStrategyMap` visualization
8. Update all hooks to new endpoints
9. Run parallel acceptance testing with both old and new APIs active

### Phase 4 — Cleanup

1. Deprecate old API endpoints (retained for one full release cycle with deprecation warnings in responses)
2. After deprecation period, drop: `perspective`, `goal`, `initiative`, `initiative_impact_on_goal`, `goal_indicators`, `goal_risks`, `initiative_milestones`, `initiative_progress_log`, `initiative_budget_sources`, `payment`
3. Remove old controllers, services, repositories, and domain entities

### Rollback Plan

- **Phases 1–2**: Fully reversible. Old tables untouched. Revert by disabling new API endpoints via feature flag.
- **Phase 3**: Frontend-reversible by reverting frontend deployment. Backend remains compatible with both clients during transition.
- **Phase 4**: Irreversible. A verified full database backup must be taken before Phase 4 execution. No rollback after Phase 4.

---

*Document maintained by the Advance360 engineering team.*
*Version 3.2 — Adds strategy-start type defaults with per-strategy overrides, strategy templates, project linkability, same-strategy multi-parent support, DAG-safe calculations, and imported-element permission coverage.*
*Implementation begins only after business and management approval.*
