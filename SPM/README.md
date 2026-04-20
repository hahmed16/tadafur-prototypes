# Advance360 — SPM Module Redesign

## Strategic Performance Management: Graph-Based Architecture

**Version:** 1.0  
**Date:** 2026-04-20  
**Status:** Design Finalized

## TABLE OF CONTENTS

1. [Overview & Approach](#1-overview--approach)
2. [User Stories](#2-user-stories)
3. [Core Concepts](#3-core-concepts)
4. [Element Types & Capabilities](#4-element-types--capabilities)
5. [Strategy Configuration](#5-strategy-configuration)
6. [The Graph Model](#6-the-graph-model)
7. [Status & Data Flow](#7-status--data-flow)
8. [KPIs & Risks](#8-kpis--risks)
9. [Vision Alignment](#9-vision-alignment)
10. [Business Rules](#10-business-rules)
11. [Validation Rules](#11-validation-rules)
12. [Calculation Principles](#12-calculation-principles)
13. [Permissions Model](#13-permissions-model)
14. [Navigation Model](#14-navigation-model)
15. [Integration Map](#15-integration-map)
16. [Open Questions](#16-open-questions)
17. [Glossary](#17-glossary)

---

## 1. OVERVIEW & APPROACH

### Why This Change

The current SPM module enforces a fixed tree hierarchy: Strategy → Perspective → Goal → Initiative. This does not reflect how organizations actually work:

- The same Goal may serve multiple Strategies
- An Initiative may contribute to several Goals across different Strategies
- Element types vary by organization — some use "Programs", "Pillars", or "Objectives"
- The rigid parent-child model leads to duplicate records and reporting gaps

### The New Approach

**1. Generic Elements with Typed Identity** — Every SPM building block is a single generic entity called an **SPM Element**. Its type is a configuration value defined at the organization level, not hardcoded.

**2. Directed Weighted Graph Structure** — Elements are connected through directed, weighted edges. A single element may have zero, one, or multiple parents in one strategy, and can participate in multiple strategies simultaneously.

**3. Strategy as a Structured Scope** — A strategy owns its edges, its type-level effective settings, and its template lineage — but not the elements themselves. Elements have independent identity; strategies reference and structure them.

### What Does Not Change

- Indicators and Risks remain specialized entities with domain-specific data
- The Vision module remains fully separate
- Soft deletion is used everywhere — no hard deletes
- All existing business features (budget tracking, progress tracking, team members, workflow) are preserved

---

## 2. USER STORIES

User stories are organized by epic. Each story includes a brief description and its acceptance criteria. Stories serve as the basis for UAT sign-off.

---

### Epic 1 — Module Setup

**US-01 — Define Element Types**  
*As a Module Admin, I want to define the element types used in my organization so that strategies are built using our terminology.*

- Admin can create up to 6 element types (4 standard + 2 custom)
- Each type has a name (localized), icon, color, and default capabilities
- Types cannot be deleted once elements of that type exist
- A type's code must be unique within the organization

---

**US-02 — Create a Strategy Template**  
*As a Module Admin, I want to create strategy templates so that new strategies can start with a pre-configured structure.*

- Admin can define element types, allowed parent-child combinations, default capabilities, and weight methods in a template
- Template can be named, described, and categorized
- Template is reusable across multiple strategies
- Template does not contain live elements — only configuration rules

---

**US-03 — Override Type Capabilities Per Strategy**  
*As a Strategy Owner or Editor, I want to adjust which capabilities are active for each element type in my strategy so that I can tailor the strategy to its context without changing the organization-wide defaults.*

- Owner/Editor can toggle capabilities (KPIs, Risks, Project Link, Quarterly Measurable) per element type within their strategy
- Changes apply only within that strategy; the organization-level type defaults are unaffected
- For strategies created from a template, these settings are pre-applied and cannot be changed during creation, but may be changed after

---

### Epic 2 — Strategy Lifecycle

**US-04 — Create a Strategy**  
*As any module user, I want to create a new strategy so that I can define and track a strategic plan.*

- User can create a strategy from scratch or from an existing template
- Strategy requires a name and at least one element type selected before it can be activated
- Creator becomes the strategy owner by default

---

**US-05 — Activate a Strategy**  
*As a Strategy Owner or Editor, I want to activate my strategy so that tracking, progress recording, and calculations become operational.*

- Strategy must have at least one element linked and a type-level configuration set before activation
- Once activated, tracking of elements within it begins according to each element's own status
- System notifies strategy team members upon activation

---

**US-06 — Suspend a Strategy**  
*As a Strategy Owner or Module Admin, I want to temporarily suspend a strategy so that no new data is recorded during a review or audit period.*

- Suspension makes all data within the strategy read-only
- No new progress entries, payments, element additions, or link changes are allowed while suspended
- Strategy can be resumed back to Active

---

**US-07 — Complete a Strategy**  
*As a Strategy Owner or Module Admin, I want to mark a strategy as completed so that its period is formally closed and data is preserved.*

- Completing a strategy freezes all data within it in this context
- Element statuses within the strategy are not automatically changed
- Strategy becomes read-only after completion

---

**US-08 — Archive a Strategy**  
*As a Module Admin, I want to archive a strategy so that it moves to long-term read-only historical storage.*

- Only Module Admin can archive
- Archived strategies cannot be modified — historical access only
- Archiving is a terminal action; there is no unarchive

---

### Epic 3 — Element Management

**US-09 — Create an Element**  
*As any module user, I want to create a new SPM element so that it can be used in strategies.*

- Any module user can create an element
- Creator becomes the element owner by default
- Element requires a name in the default organization language
- New elements start in DRAFT status
- Element can exist without being linked to any strategy

---

**US-10 — Edit an Element**  
*As an Element Owner or Editor, I want to edit an element's details so that its information stays current.*

- Owner and Editor can edit name, description, dates, organization unit, and team members while the element is in DRAFT or ACTIVE status
- Start date cannot be changed once the element has been activated
- Editing an element's global fields affects its appearance across all strategies it belongs to

---

**US-11 — Activate an Element**  
*As an Element Owner or Editor, I want to activate an element so that it becomes operational and contributes to strategy calculations.*

- Element must have a name set in the default organization language
- If the element type is Trackable, a start date is required
- If the element type is Budgetable, a planned budget is recommended (warning shown if missing; activation is not blocked)
- Once activated, the start date is permanently locked
- Activation triggers recalculation of all parent elements in every strategy this element belongs to
- Team members and parent element owners are notified

---

**US-12 — Put an Element On Hold**  
*As an Element Owner, Editor, or any Strategy Editor of a containing strategy, I want to temporarily pause an element so that it is suspended without losing its contributions to parents.*

- Element's progress is frozen at its last recorded value
- No new progress entries can be added while on hold
- The frozen progress value continues to count toward parent calculations
- Committed payments may still be recorded while on hold
- Element can be resumed back to Active

---

**US-13 — Resume an Element**  
*As an Element Owner or Editor, I want to resume an element from hold so that tracking continues.*

- All editing and tracking rights are restored to Active rules
- Expected progress recalculates to account for the hold period
- Parent elements recalculate immediately

---

**US-14 — Complete an Element**  
*As an Element Owner or Editor, I want to mark an element as completed so that its final progress is recorded and the element is formally closed.*

- At least one progress log entry must exist if the element type is Trackable
- An alert is shown if consumed budget is less than allocated budget (not a blocker)
- All fields become read-only after completion
- End date is set to today if not already provided
- For Budgetable elements, a 30-day grace period allows final payment reconciliation
- The element's final progress permanently contributes to parent calculations
- Team members and parent owners are notified

---

**US-15 — Deactivate an Element**  
*As an Element Owner or Module Admin, I want to deactivate an element so that it is suspended indefinitely and removed from all calculations.*

- System shows a warning listing all parent elements across all strategies that will be affected before proceeding
- User must confirm before deactivation completes
- Element is removed from all parent calculations; edge weight is treated as zero
- Children of the deactivated element are not automatically deactivated
- Element can be reactivated later

---

**US-16 — Reactivate an Element**  
*As an Element Owner or Module Admin, I want to reactivate a deactivated element so that it rejoins strategy calculations.*

- All editing rights are restored
- Element re-enters parent calculations in all strategies; affected parents recalculate
- The gap period during deactivation has no progress entries

---

**US-17 — Deprecate an Element**  
*As an Element Owner, Module Admin, or Strategy Owner of a containing strategy, I want to permanently archive an element so that it is no longer usable.*

- Element must be in DRAFT, COMPLETED, or DEACTIVATED status — ACTIVE and ON_HOLD elements cannot be deprecated directly
- Deprecated elements are hidden from all default views; visible only in archive/history when explicitly filtered
- Deprecated elements cannot be linked to any new strategy
- Deprecation is terminal — there is no way to reverse it

---

### Epic 4 — Graph Building

**US-18 — Link Elements in a Strategy**  
*As a Strategy Owner or Editor, I want to link elements as parent-child within my strategy so that the strategy map reflects the contribution hierarchy.*

- Owner/Editor can add an element as a child under any other element, subject to the strategy's type-level configuration
- If the link does not conform to the configured hierarchy, a visual warning is shown — but the link is not blocked
- Cycles are not allowed; the system prevents linking a descendant as a parent

---

**US-19 — Set Edge Weights**  
*As a Strategy Owner or Editor, I want to set how much each child element contributes to its parent so that progress and budget roll up accurately.*

- Supported weight methods: Percentage (0–100%), Impact score (1–10), Equal (auto-distributed), Business Value (derived from child's business value field), or Budget-based (derived from child's planned budget)
- A strategy-level default weight method is set during configuration and applies to all new edges; it can be overridden per individual edge

---

**US-20 — Link an Element to Multiple Parents**  
*As a Strategy Owner or Editor, I want to link one element under multiple parents in the same strategy so that shared contributions are modeled without duplication.*

- An element can have multiple parents in the same strategy as long as no cycle is created
- When a child has multiple parents, allocation percentages must be set across those parents and must sum to 100%
- This ensures the child's progress and budget are split proportionally — never counted in full under every parent

---

**US-21 — Remove an Element from a Strategy**  
*As a Strategy Owner or Editor, I want to remove an element from a strategy without deleting it so that it remains available for use in other strategies.*

- Removing an element from a strategy removes only the edges connecting it within that strategy
- The element itself remains in the organization's element library
- Parent elements in the strategy recalculate after the removal

---

### Epic 5 — Progress Tracking

**US-22 — Log Progress**  
*As an Element Owner or Editor, I want to log a progress entry so that execution progress is recorded over time.*

- Progress entries can only be logged on Active elements whose type has the Trackable capability
- Progress rate must be between 0% and 100%
- Entry date cannot be in the future
- Logging a new entry updates the element's own progress and triggers recalculation of all parent elements

---

**US-23 — Track Milestones**  
*As an Element Owner or Editor, I want to add and complete milestones so that key delivery checkpoints are tracked.*

- Milestones can only be added and completed on Active Trackable elements
- Milestones should fall within the element's start and end date range (warning shown if outside)

---

**US-24 — View Progress Discrepancy**  
*As an Element Owner or any viewer, I want to see when my manually entered progress differs significantly from the system-calculated progress so that I can investigate whether my estimate is accurate.*

- The system displays both the manually entered progress (Own Progress) and the calculated progress derived from children
- A discrepancy indicator is shown when the difference exceeds the configured threshold (default 5%)
- The discrepancy alert does not block any action — it prompts the owner to review

---

### Epic 6 — Budget Management

**US-25 — Set a Planned Budget**  
*As an Element Owner or Editor, I want to set a planned budget for a budgetable element so that financial planning is captured.*

- Planned budget can be set on Draft or Active elements whose type has the Budgetable capability
- Currency type is locked once the element is activated
- Planned budget is a global field — it is the same across all strategies this element appears in

---

**US-26 — Record Payments**  
*As an Element Owner or Editor, I want to record actual payment disbursements so that consumed budget is tracked.*

- Payments can only be recorded on Active or On Hold Budgetable elements
- Payment amount must be positive
- Completed elements allow payments within a 30-day grace period after completion

---

**US-27 — View Aggregated Budget**  
*As any viewer with access, I want to see the aggregated planned, allocated, and consumed budget at any level of the strategy hierarchy so that financial performance is visible.*

- Budget totals at a parent level sum all descendants in that strategy, excluding Deactivated and Deprecated elements
- When an element has multiple parents, its budget is split proportionally — it is never counted in full under every parent

---

**US-28 — Auto-Calculated Allocated Budget**  
*As a Strategy Owner, I want allocated budgets to be automatically derived from contribution percentages so that budget distribution is always consistent with the strategy weights and does not need to be entered manually.*

- Allocated budget is not entered by users — it is calculated by the system
- For each parent, the system sums all outgoing contribution percentages, then assigns each child's allocated budget as a proportional share of the parent's planned budget
- If contribution percentages do not sum to 100% (e.g., weights are 70% and 70%), the system normalizes them automatically before applying

---

### Epic 7 — KPI & Risk Linking

**US-29 — Link a KPI to an Element**  
*As a Strategy Owner or Editor, I want to link KPIs to elements so that measurable targets are associated with strategic goals.*

- KPI links are strategy-scoped — the same KPI can be linked to different elements in different strategies
- A KPI can only be linked once to the same element in the same strategy
- KPI links can only be created when the strategy is Draft or Active and the element is not Completed, Deactivated, or Deprecated

---

**US-30 — Link a Risk to an Element**  
*As a Strategy Owner or Editor, I want to link risks to elements so that threats are associated with the goals or initiatives they affect.*

- Risk links follow the same scoping and creation rules as KPI links
- A risk can only be linked once to the same element in the same strategy

---

**US-31 — View Element Health Score**  
*As any viewer, I want to see an element's health score so that I can quickly assess how well its linked KPIs are being met.*

- Health score = percentage of linked KPIs meeting their current period targets, weighted by KPI contribution weight
- A health score is only shown when at least one Active KPI is linked
- Health score is visible on Active, On Hold, and Completed elements

---

### Epic 8 — Element Sharing & Reuse

**US-32 — Share an Element**  
*As an Element Owner, I want to share my element so that other strategy owners and editors can import and use it in their strategies.*

- Sharing is controlled by the element owner or Module Admin
- Sharing does not transfer ownership — the source owner retains full control
- A shared element appears in the shared element library for discovery by other teams

---

**US-33 — Import a Shared Element**  
*As a Strategy Owner or Editor, I want to import a shared element into my strategy so that I can reuse existing work without creating duplicate records.*

- Importer must have an active share grant or be granted import access by the source owner
- Importing grants strategy-scoped usage rights only — the importer cannot edit the element's global fields (name, description, owner)
- The element appears in the strategy alongside owned elements; its global identity remains unchanged

---

**US-34 — Revoke Element Sharing**  
*As an Element Owner, I want to revoke sharing on my element so that it can no longer be imported by other teams.*

- Revoking sharing blocks all future imports
- Elements already imported into other strategies remain usable and readable by those teams
- Source ownership and global edit rights remain with the original owner's team

---

**US-35 — See Where an Element Is Used**  
*As any user with view access to an element, I want to see which strategies this element participates in so that I understand its full usage context before making changes.*

- A "used in X strategies" indicator is shown on the element's detail page and in the element library
- Listing shows the strategy names and the user's access level in each

---

### Epic 9 — Access & Permissions

**US-36 — Request Access to an Element Type**  
*As any module user, I want to request access to a specific element type within a strategy so that I can view or contribute to those elements without the owner having to assign me individually.*

- User submits a request specifying: target strategy, target element type, and requested role (Viewer or Editor)
- Request is routed to the Module Admin or the Strategy Owner of the target strategy
- Approver can approve or deny; user is notified either way
- Approved access applies to all elements of the requested type within the specified strategy scope
- Access follows the same top-down inheritance rules as direct assignments

---

**US-37 — Approve or Deny an Access Request**  
*As a Module Admin or Strategy Owner, I want to review and act on access requests so that access is granted deliberately and auditably.*

- Approver sees the requester, requested scope, element type, and role level
- Approving writes the permission in the authorization system for the specified scope
- All approvals and denials are audited (approver, timestamp, granted scope)

---

**US-38 — Assign Team Members to an Element**  
*As an Element Owner, I want to assign team members to my element so that the right people can view or edit it.*

- Owner can assign users as Viewer or Editor on the element
- Assignments are global — they apply to this element across all strategies
- Team members can be added or removed while the element is in Draft, Active, or On Hold status

---

**US-39 — Inherit Access Through Strategy Structure**  
*As a Strategy Owner, I want team members assigned to a parent element to automatically inherit access to child elements so that I do not have to assign access at every level of the hierarchy.*

- If a user is assigned as Editor of a parent element, they inherit strategy-scoped access to all child elements under that parent in the same strategy
- Inheritance is computed through the strategy graph — it does not grant global element rights
- If the parent-child link is removed, inherited access from that path is revoked (unless another active parent path still grants it)
- If an element has multiple parents, inherited access is the union of all active parent paths

---

**US-40 — Activate All Elements in a Strategy at Once**  
*As a Strategy Owner or Module Admin, I want to activate all Draft elements in a strategy in a single action so that I do not have to activate them one by one when launching a strategy.*

- Bulk activation is available to Strategy Owner and Module Admin
- Each element's individual pre-conditions (name, start date for Trackable) must be met — elements that fail their pre-conditions are skipped with a per-element reason listed
- Successfully activated elements immediately join all parent calculations

---

## 3. CORE CONCEPTS

### SPM Element

A single generic entity representing any strategic building block, identified by a **type** defined at the organization level.

**Base fields shared by all elements:**

- Name (localized), Description (localized)
- Start Date / End Date
- Workflow Status
- Responsible Organization Unit
- Audit trail (created by, modified by, deleted by with timestamps)

Additional fields are unlocked by the element type's declared capabilities. Capabilities at the organization level define the **default starting profile** for the type. When that type is added to a strategy, defaults are copied into the strategy configuration and may then be overridden without changing the organization-level type.

### Element Type

A configuration record at the organization level defining:

- The type name (e.g., "Strategic Perspective", "Goal", "Initiative")
- Its default capabilities
- Its visual representation (icon, color)
- Its position in the default display order

Types are not hardcoded. Organizations define their own. A maximum of **6 types per organization** is enforced (4 standard + 2 custom). An element type defines starting defaults — strategy owners/editors can accept or override them per strategy.

### Strategy

A strategic plan referencing a set of elements with directed, weighted connections in a specific context. A strategy owns its **edges**, its type-level effective settings, and its template lineage — not the elements themselves. A strategy has its own independent lifecycle status.

### Strategy Template

A reusable blueprint predefining element types, allowed parent/child combinations, effective capability toggles, weight methods, and validation rules. Creating a strategy from a template copies those settings; the strategy may then diverge after instantiation.

### Edge (Link)

A directed, weighted connection between two elements within a strategy.

- **Direction**: Parent → Child
- **Weight**: Contribution of the child to the parent's aggregated metrics
- **Scope**: Every edge belongs to exactly one strategy
- **Multiplicity**: The same child can be linked to multiple parents within the same strategy, provided no cycle is created

---

## 4. ELEMENT TYPES & CAPABILITIES

### Standard Types

Organizations configure up to 4 standard types (e.g., Perspective, Goal, Program, Initiative) plus up to 2 custom types.

### Capabilities

| Capability                | What It Unlocks                                                                | Notes                                            |
| ------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------ |
| **Budgetable**            | Planned budget, allocated budget, budget sources, payment disbursement records | Covers both budget planning and payment tracking |
| **Trackable**             | Progress log entries, milestone entries, own progress + calculated progress    | Independent of Budgetable                        |
| **Has KPIs**              | KPI tab, KPI link actions, KPI contribution settings                           | Default copied into strategy type configuration  |
| **Has Risks**             | Risk tab, risk link actions, mitigation workflows                              | Default copied into strategy type configuration  |
| **Linkable With Project** | Project-link section and project association actions                           | Can be enabled/disabled per strategy             |
| **VisionAligned**         | Vision alignment field (Vision Priority, Vision Goals, Vision Indicators)      | Links element to a national/org vision priority  |
| **Quarterly Measurable**  | Quarterly target/actual tab and threshold-driven alerts                        | Default copied into strategy type configuration  |

### Universal Features (all elements)

- **Team Members**: Any element can have team members assigned with roles (owner, editor, viewer)
- **Status Lifecycle**: All elements follow the same status flow regardless of type

### Capability Rules

- Organization-level type capabilities provide defaults; strategy-level configuration stores the effective values for that strategy
- All capabilities may differ from one strategy to another for the same element type
- **VisionAligned** remains independent and optional for any type

---

## 5. STRATEGY CONFIGURATION & TEMPLATES

### Strategy Creation Modes

- **From scratch**: strategy owners/editors can change all strategy-level type settings including capabilities, root eligibility, parent rules, and edge weighting methods
- **From template**: template-defined type settings are pre-applied during creation and not editable there; the strategy may diverge after instantiation

### Strategy Templates

Each template includes:

- Template name, description, category, and version
- Selected element types and their effective per-strategy capability defaults
- Root eligibility, allowed parent type combinations
- Default and pair-level edge weight methods
- Strict hierarchy setting, discrepancy threshold
- Optional recommended navigation profile

### Type Level Configuration

Each strategy defines which element types it uses, their allowed parent combinations, and the effective defaults per type. **This configuration is the authoritative build specification for the strategy.** Links outside the configured hierarchy are flagged visually but are not hard-blocked.

### Default Weight Input Type

- **Percentage**: users enter 0–100%
- **Impact Score**: users enter 1–10, normalized relative to siblings under the same parent

This default can be overridden per individual edge.

---

## 6. THE GRAPH MODEL

### Structure

Each element appears **exactly once as a node** within a strategy but may have zero, one, or multiple parent edges. The same element can be linked to different strategies under different parent sets in each.

Key rules:

- Each element appears once per strategy as a node
- A node may be linked under multiple parents in the same strategy
- Edge weights are independent per edge and per strategy
- Draft and Deactivated children are excluded from parent progress calculations

### Edge Weight — Two Separate Concerns

Every edge carries two distinct settings:

- **Allocation Percent** — how much of the child's owned value is assigned to this specific parent. When a child has multiple parents, allocation percentages across all parents must sum to 100%. This prevents double-counting.
- **Contribution Weight** — how strongly this child contributes relative to its siblings under this one parent. This controls local rollup behavior and is independent of how many other parents the child has.

These two settings solve different problems and cannot be collapsed into one field.

**Supported weight methods per edge:** Percentage, Impact Score, Equal (auto-distributed), Business Value (derived from child's business value), Budget-based (derived from child's planned budget)

### Shared Elements (Cross-Strategy Reuse)

When the same element is used in multiple strategies, it is marked with a **"used in X strategies" indicator**. The element's global data (name, description, status) is identical in all strategies; only the strategy context (parents, weights, effective type settings) differs per strategy.

---

## 7. STATUS & DATA FLOW

This section defines the complete lifecycle of every SPM element and strategy — statuses, allowed transitions, who can trigger each transition, pre-conditions, post-conditions, and calculation impacts.

---

### 7.1 Element Status Lifecycle

#### Status Definitions

| Status          | Meaning                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------------------- |
| **DRAFT**       | Being prepared. Not operational. Editable freely. Excluded from parent calculations.                          |
| **ACTIVE**      | Live and operational. Tracking has begun. Included in all calculations.                                       |
| **ON_HOLD**     | Temporarily paused. Progress frozen at last value. Still counted in parent calculations at frozen value.      |
| **COMPLETED**   | Finished successfully. All data frozen. Contributes final progress to parents permanently.                    |
| **DEACTIVATED** | Suspended indefinitely. Excluded from all calculations. Can be reactivated.                                   |
| **DEPRECATED**  | Obsolete. Permanently excluded from calculations. Cannot be linked to new strategies. **Terminal — no exit.** |

#### State Diagram

```
DRAFT ──────→ ACTIVE ──────→ ON_HOLD ──────→ ACTIVE (resume)
  │               │               │
  │               │               └──────→ DEACTIVATED ──────→ DEPRECATED
  │               │                               ↑
  │               ├──────→ COMPLETED ──────→ DEPRECATED
  │               │
  │               └──────→ DEACTIVATED
  │
  └──────→ DEPRECATED (cancelled before activation)
```

#### Forbidden Transitions

- COMPLETED → ACTIVE (cannot reverse completion)
- DEPRECATED → any status (no exit from terminal state)
- ON_HOLD → COMPLETED (must resume to ACTIVE first)
- Any transition not listed in the table below is rejected

---

#### Element Status Transition Table

| From        | To          | Who Can Trigger                                                          | Pre-Conditions                                                                                                                                | Post-Conditions & Calculation Impact                                                                                                                                                                                                                                                 |
| ----------- | ----------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| DRAFT       | ACTIVE      | Element Owner, Element Editor                                            | Name set in default org language. If Trackable: start date required. If Budgetable: planned budget recommended (warning only, not a blocker). | Start date permanently locked. Progress tracking begins (if Trackable). Element included in all parent calculations in every strategy it belongs to. Parents recalculate upward. Team members and parent owners notified.                                                            |
| DRAFT       | DEPRECATED  | Element Owner, Module Admin                                              | Element must not have been activated.                                                                                                         | Permanently excluded from calculations. Cannot be linked to any new strategy. Hidden from default views. Terminal state.                                                                                                                                                             |
| ACTIVE      | ON_HOLD     | Element Owner, Element Editor, Strategy Editor (any containing strategy) | None.                                                                                                                                         | Progress frozen at last recorded value. No new progress entries. Milestones suspended. Committed payments still allowed. Element remains in parent calculations using frozen value. Team members and parent owners notified.                                                         |
| ACTIVE      | COMPLETED   | Element Owner, Element Editor                                            | If Trackable: at least one progress log entry must exist. If Budgetable: alert shown if consumed < allocated (warning only, not a blocker).   | All fields locked (read-only). End date set to today if not already provided. Progress fixed at 100% (or last logged value if lower). 30-day payment grace period for Budgetable. Final frozen progress permanently contributes to parents. Team members and parent owners notified. |
| ACTIVE      | DEACTIVATED | Element Owner, Module Admin                                              | System shows warning listing all affected parent elements across all strategies. User must confirm.                                           | All fields immediately read-only. Element removed from all parent calculations. Children are not automatically deactivated. All affected parents recalculate upward across all strategies. Team members and all parent owners in all strategies notified.                            |
| ON_HOLD     | ACTIVE      | Element Owner, Element Editor                                            | None.                                                                                                                                         | All editing rights restored. Progress tracking resumes; expected progress recalculates for the hold period. Element rejoins active calculations; parents recalculate immediately. Team members and parent owners notified.                                                           |
| ON_HOLD     | DEACTIVATED | Element Owner, Module Admin                                              | Same warning as ACTIVE → DEACTIVATED.                                                                                                         | Same post-conditions as ACTIVE → DEACTIVATED.                                                                                                                                                                                                                                        |
| COMPLETED   | DEPRECATED  | Module Admin, Strategy Owner (any containing strategy)                   | Element must be in COMPLETED state.                                                                                                           | Permanently archived. Cannot be linked to new strategies. Hidden from all default views; visible only in archive/history views. Team members notified.                                                                                                                               |
| DEACTIVATED | ACTIVE      | Element Owner, Module Admin                                              | None.                                                                                                                                         | Editing rights restored. Element re-enters all parent calculations; parents recalculate. Progress gap period during deactivation has no entries; expected progress adjusts. Team members and parent owners notified.                                                                 |
| DEACTIVATED | DEPRECATED  | Module Admin                                                             | None.                                                                                                                                         | Terminal state. Permanently excluded from calculations and linking. Hidden from default views.                                                                                                                                                                                       |

---

### 7.2 What Changes at Each Status Transition

**DRAFT → ACTIVE**

| Area                  | Change                                                                                                                 |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **All elements**      | Start date locked permanently. Status becomes globally Active across all strategies containing this element.           |
| **Budgetable**        | Planned budget locked for currency type. Budget fields visible in financial dashboards and parent budget aggregations. |
| **Trackable**         | Progress tracking begins. Progress log entries can be created. Milestones become actionable.                           |
| **VisionAligned**     | Vision alignment becomes visible in Vision module alignment reports.                                                   |
| **Indicators linked** | Target tracking begins. Actuals measured against targets. Indicator contributions to health score active.              |
| **Risks linked**      | Risk monitoring starts. Risk phase progression tracking begins.                                                        |
| **Calculations**      | Element included in all parent progress calculations in every strategy it belongs to. Parents recalculate upward.      |
| **Notifications**     | Team members and parent element owners notified.                                                                       |

---

**ACTIVE → ON_HOLD**

| Area                  | Change                                                                                         |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| **All elements**      | Description and end date extension editable. All other fields locked.                          |
| **Budgetable**        | No new planned budget changes. Committed amounts adjustable. Committed payments still allowed. |
| **Trackable**         | Progress frozen at last recorded values. No new entries. Milestones suspended.                 |
| **Indicators linked** | Actuals can still be entered for monitoring continuity. Targets unchanged.                     |
| **Risks linked**      | Risk monitoring and phase progression continue.                                                |
| **Calculations**      | Element remains in parent calculations using frozen progress value.                            |

---

**ON_HOLD → ACTIVE (Resume)**

| Area             | Change                                                                                                     |
| ---------------- | ---------------------------------------------------------------------------------------------------------- |
| **All elements** | All editing rights restored.                                                                               |
| **Trackable**    | Progress tracking resumes. New entries can be created. Expected progress recalculates for the hold period. |
| **Calculations** | Element rejoins active weighted calculations. Parent elements recalculate immediately.                     |

---

**ACTIVE → COMPLETED**

| Area                  | Change                                                                                                    |
| --------------------- | --------------------------------------------------------------------------------------------------------- |
| **All elements**      | All fields locked (read-only). End date set to today if not already provided.                             |
| **Budgetable**        | No budget changes. 30-day payment grace period for final reconciliation; after that, all payments locked. |
| **Trackable**         | Own progress fixed at 100% (or last logged value if lower). No new progress entries.                      |
| **Indicators linked** | Final period actuals can still be recorded for the reporting period.                                      |
| **Risks linked**      | Open risk phases must be resolved, transferred, or accepted. No new risk assessments.                     |
| **Calculations**      | Element contributes final frozen progress to parent calculations permanently.                             |

---

**ACTIVE → DEACTIVATED**

| Area             | Change                                                                                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All elements** | All fields immediately read-only.                                                                                                                    |
| **Calculations** | Element removed from all parent calculations. Parents recalculate upward through every strategy that contains this element.                          |
| **Children**     | Children are not automatically deactivated. A warning lists all children and all affected parents across all strategies before the action completes. |

---

**DEACTIVATED → ACTIVE (Reactivate)**

| Area             | Change                                                                                                                       |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Trackable**    | Progress tracking resumes. Gap period during deactivation has no entries. Expected progress adjusts for the deactivated gap. |
| **Calculations** | Element re-enters all parent calculations. Affected parents recalculate.                                                     |

---

**Any Status → DEPRECATED (Terminal)**

| Area             | Change                                                                                         |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| **All elements** | Permanently read-only.                                                                         |
| **Calculations** | Excluded from all calculations. Affected parents recalculate.                                  |
| **Linking**      | Cannot be linked to any new strategy. Cannot receive new indicator or risk links.              |
| **Visibility**   | Hidden from all default views. Visible only in archive/history views when explicitly filtered. |

---

### 7.3 Strategy Status Lifecycle

#### Strategy Status Definitions

| Status        | Meaning                                                                                       |
| ------------- | --------------------------------------------------------------------------------------------- |
| **DRAFT**     | Being configured. Elements can be added and linked. Progress and payments cannot be recorded. |
| **ACTIVE**    | Live. All tracking, calculations, and reporting operational.                                  |
| **SUSPENDED** | Temporarily paused. No new tracking. Existing data read-only in this strategy context.        |
| **COMPLETED** | Period ended. All data frozen in this context. Historical access only.                        |
| **ARCHIVED**  | Permanently archived. Read-only historical reference. **Terminal — no exit.**                 |

#### State Diagram

```
DRAFT ──────→ ACTIVE ──────→ SUSPENDED ──────→ ACTIVE (resume)
  │               │
  │               ├──────→ COMPLETED ──────→ ARCHIVED
  │               └──────→ ARCHIVED (abandoned)
  │
  └──────→ ARCHIVED (abandoned before activation)
```

#### Strategy Status Transition Table

| From      | To        | Who Can Trigger                 | Pre-Conditions                                                | Post-Conditions                                                                          |
| --------- | --------- | ------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| DRAFT     | ACTIVE    | Strategy Owner, Strategy Editor | At least one element linked. Type level configuration is set. | All tracking, calculations, and reporting become operational.                            |
| ACTIVE    | SUSPENDED | Strategy Owner, Module Admin    | None.                                                         | No new tracking, progress entries, payments, element additions, or link changes allowed. |
| ACTIVE    | COMPLETED | Strategy Owner, Module Admin    | None.                                                         | All data frozen in this strategy context. Element statuses are not changed.              |
| ACTIVE    | ARCHIVED  | Module Admin                    | None.                                                         | Permanently archived. Read-only historical access. Terminal state.                       |
| SUSPENDED | ACTIVE    | Strategy Owner, Module Admin    | None.                                                         | All element operations resume per each element's own status rules.                       |
| SUSPENDED | ARCHIVED  | Module Admin                    | None.                                                         | Terminal state.                                                                          |
| COMPLETED | ARCHIVED  | Module Admin                    | None.                                                         | Terminal state.                                                                          |
| DRAFT     | ARCHIVED  | Module Admin                    | None.                                                         | Terminal state (abandoned before activation).                                            |

#### How Strategy Status Affects Element Operations

| Strategy Status | Element Operations Allowed                                                            |
| --------------- | ------------------------------------------------------------------------------------- |
| **DRAFT**       | Add/remove elements and links. Edit element data. Cannot record progress or payments. |
| **ACTIVE**      | All operations allowed per each element's own status rules.                           |
| **SUSPENDED**   | Read-only. No new progress entries or payments. No new element or link additions.     |
| **COMPLETED**   | Read-only. No changes to any element within this strategy context.                    |
| **ARCHIVED**    | Read-only historical access. No changes.                                              |

---

### 7.4 Data Flow Summary by Capability

#### Budgetable Elements

| Operation                             | DRAFT | ACTIVE | ON_HOLD            | COMPLETED        | DEACTIVATED | DEPRECATED |
| ------------------------------------- | ----- | ------ | ------------------ | ---------------- | ----------- | ---------- |
| Set planned budget                    | ✅     | ✅      | ❌                  | ❌                | ❌           | ❌          |
| Update allocated budget               | ✅     | ✅      | ✅ (committed only) | ❌                | ❌           | ❌          |
| Add budget source                     | ✅     | ✅      | ❌                  | ❌                | ❌           | ❌          |
| Create payment record                 | ❌     | ✅      | ✅ (committed only) | ✅ (grace period) | ❌           | ❌          |
| Edit payment                          | ❌     | ✅      | ✅                  | ✅ (grace period) | ❌           | ❌          |
| View budget & payments                | ✅     | ✅      | ✅                  | ✅                | ✅           | ✅          |
| Included in parent budget aggregation | ❌     | ✅      | ✅                  | ✅                | ❌           | ❌          |

#### Trackable Elements

| Operation                               | DRAFT | ACTIVE | ON_HOLD    | COMPLETED  | DEACTIVATED | DEPRECATED |
| --------------------------------------- | ----- | ------ | ---------- | ---------- | ----------- | ---------- |
| Add progress log entry                  | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| Add milestone                           | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| Mark milestone complete                 | ❌     | ✅      | ❌          | ❌          | ❌           | ❌          |
| View own progress                       | ✅     | ✅      | ✅          | ✅          | ✅           | ✅          |
| View calculated progress                | ✅     | ✅      | ✅          | ✅          | ✅           | ✅          |
| View discrepancy indicator              | —     | ✅      | ✅          | ✅          | —           | —          |
| Included in parent progress calculation | ❌     | ✅      | ✅ (frozen) | ✅ (frozen) | ❌           | ❌          |

#### Indicators Linked to Element

| Operation                             | DRAFT | ACTIVE | ON_HOLD        | COMPLETED            | DEACTIVATED | DEPRECATED |
| ------------------------------------- | ----- | ------ | -------------- | -------------------- | ----------- | ---------- |
| Link new indicator                    | ✅     | ✅      | ❌              | ❌                    | ❌           | ❌          |
| Unlink indicator                      | ✅     | ✅      | ❌              | ❌                    | ❌           | ❌          |
| Enter actual values for indicator     | ❌     | ✅      | ✅ (monitoring) | ✅ (reporting period) | ❌           | ❌          |
| Indicator contributes to health score | ❌     | ✅      | ✅              | ✅                    | ❌           | ❌          |

#### Risks Linked to Element

| Operation                      | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED | DEPRECATED |
| ------------------------------ | ----- | ------ | ------- | --------- | ----------- | ---------- |
| Link new risk                  | ✅     | ✅      | ❌       | ❌         | ❌           | ❌          |
| Unlink risk                    | ✅     | ✅      | ❌       | ❌         | ❌           | ❌          |
| Progress risk phase            | ❌     | ✅      | ✅       | ❌         | ❌           | ❌          |
| Risk contributes to risk score | ❌     | ✅      | ✅       | ✅         | ❌           | ❌          |

#### Team Members (all elements)

| Operation          | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED | DEPRECATED |
| ------------------ | ----- | ------ | ------- | --------- | ----------- | ---------- |
| Add team member    | ✅     | ✅      | ✅       | ❌         | ❌           | ❌          |
| Remove team member | ✅     | ✅      | ✅       | ❌         | ❌           | ❌          |
| View team members  | ✅     | ✅      | ✅       | ✅         | ✅           | ✅          |

#### Graph Links (edges in a strategy)

| Operation                   | DRAFT | ACTIVE | ON_HOLD | COMPLETED | DEACTIVATED    | DEPRECATED |
| --------------------------- | ----- | ------ | ------- | --------- | -------------- | ---------- |
| Add as child to a parent    | ✅     | ✅      | ❌       | ❌         | ❌              | ❌          |
| Remove from parent (unlink) | ✅     | ✅      | ❌       | ❌         | ✅ (admin only) | ❌          |
| Update edge weight          | ✅     | ✅      | ❌       | ❌         | ❌              | ❌          |

---

### 7.5 Status Propagation to Parent Calculations

When an element's status changes, all parent elements in every strategy that contains it recalculate their derived metrics, propagating all the way to the root.

The recalculation covers: progress, health score (if indicators linked), and budget aggregation (if budgetable).

Recalculation runs asynchronously to avoid blocking the user's action. Displayed values on open screens update automatically without requiring a page reload.

---

## 8. KPIs & RISKS

Indicators and Risks are **specialized entities**, not generic SPM elements. They have domain-specific data that does not fit the generic element model.

### KPIs

Remain fully specialized with:

- Time-series measurement data (baseline, target, actual by Year-Quarter)
- Measure type (Monetary, Quantity, Rate)
- Indicator type (Key Indicator, Vision Indicator, Sub-Indicator, Benchmarking, Cross-Cutting, Vertical Alignment)
- Bigger-is-better flag, absolute baseline and target
- Team member management

**Lifecycle**: DRAFT, ACTIVE, DEPRECATED.

**Linking**: Optional. An indicator can exist as a standalone entity. When linked to an element, the link is strategy-scoped and carries a weight representing the indicator's contribution to the element's health score.

### Risks

Remain fully specialized with:

- Risk category, type, urgency, escalation level
- Impact assessment (base and residual impact and probability)
- Mitigation plan with treatment decision and costs
- Mitigation phases with status and progress tracking
- Monitoring frequency and owner assignment

**Lifecycle**: IDENTIFIED, ASSESSED, MITIGATING, RESOLVED, ACCEPTED, CLOSED.

**Linking**: Optional. A risk can exist without being linked to any element. When linked, the link is strategy-scoped.

### Scope of Linking (Phased Rollout)

| Link Type                            | Phase 1 (Initial Release) | Future Release |
| ------------------------------------ | ------------------------- | -------------- |
| Indicator → Goal-type element        | ✅ Enabled                 | —              |
| Indicator → Initiative-type element  | ✅ Enabled                 | —              |
| Indicator → Perspective-type element | Built, frontend disabled  | ✅              |
| Indicator → Strategy (top-level KPI) | Built, frontend disabled  | ✅              |
| Risk → Goal-type element             | ✅ Enabled                 | —              |
| Risk → Initiative-type element       | ✅ Enabled                 | —              |
| Risk → Perspective-type element      | Built, frontend disabled  | ✅              |
| Risk → Strategy (top-level)          | Built, frontend disabled  | ✅              |

---

## 9. VISION ALIGNMENT

The Vision module (Vision Pillar → Vision Priority → Vision Goal / Vision Indicator) remains completely separate and unchanged.

SPM Elements with the **VisionAligned** capability have an optional **Vision Alignment** field. This is metadata — it shows which vision priority this element serves and does not create a structural dependency between the SPM graph and the Vision hierarchy.

Optional alignment fields per element:

- Vision Priority (select one from the Vision module)
- One or more Vision Goals within the selected Vision Priority
- One or more Vision Indicators within the selected Vision Priority

Vision alignment data is read-only in Vision module reports and can only be edited from the SPM element's detail page.

---

## 10. BUSINESS RULES

### Element Ownership & Identity

**BR-01** An element belongs to an organization. Elements cannot be shared across organizations.

**BR-02** The creator of an element is its initial owner. Any team member with manage-members permission can transfer the owner role to another user. At any time, there is exactly one owner per element.

**BR-03** An element has a single global identity. Editing its name, description, dates, or status changes it everywhere — across all strategies that contain it.

**BR-04** An element can exist without being part of any strategy and can be linked to strategies later.

**BR-05** An element can be part of multiple strategies simultaneously.

### Linking Rules

**BR-06** An element is added to a strategy either by creating an edge involving it or by explicitly adding it as a root element with no parent.

**BR-07** Within a single strategy, each element appears exactly once as a node. It may be a root node, have one parent, or have multiple parents.

**BR-08** The same element can be referenced in multiple strategies. In each strategy it has its own parent set, edge weights, and display order. The element's global data remains identical across all strategies.

**BR-09** Edges are strategy-scoped. When a strategy is soft-deleted, all its edges are removed. The elements themselves remain in the organization's element library.

**BR-10** Linking an existing element from another strategy does not change that element's data or its links in the other strategy.

**BR-11** A DEPRECATED element cannot be linked to any new strategy or receive new indicator/risk links.

**BR-12** Edges can only be created or modified when both the strategy and the target element are in an editable state (not COMPLETED, ARCHIVED, or DEPRECATED).

**BR-13** Links within a strategy should follow the strategy's type level configuration. A non-conforming link produces a visual warning but is not hard-blocked.

### Deletion Rules

**BR-14** All deletions are soft deletes. No record is permanently removed.

**BR-15** Soft-deleting an element excludes it from all views and calculations. All strategy edges are retained for historical integrity.

**BR-16** Removing an element from a strategy removes only the edges connecting it within that strategy. The element itself is unaffected.

**BR-17** Soft-deleting a strategy marks the strategy record only. All linked elements are unaffected.

**BR-18** A soft-deleted element behaves identically to DEPRECATED for calculation purposes.

**BR-19** Restoring a soft-deleted element makes it visible again in all strategies it was linked to. Requires Module Admin or Strategy Owner of any containing strategy.

### Status Rules

**BR-20** Workflow status is a global property of the element. The same status applies in all strategies.

**BR-21** Status transitions must follow the allowed map in Section 7.1. Any other transition is rejected.

**BR-22** Transitioning an element to DEACTIVATED or DEPRECATED does not automatically change the status of its children. Children continue independently.

**BR-23** When deactivating an element, the system presents a warning listing all parent elements across all strategies that will be affected.

**BR-24** DEPRECATED is a terminal state. No transition out of DEPRECATED is possible for any user.

**BR-25** A strategy cannot be activated if it has no type level configuration or no elements.

**BR-26** A strategy's COMPLETED or ARCHIVED transition does not change the status of the elements within it.

### Budget Rules

**BR-27** Planned budget is a global property of an element. It does not vary per strategy.

**BR-28** Budget fields are available only on elements whose type has the Budgetable capability.

**BR-29** Budget aggregation at parent level is context-computed: sums budgets of all descendant elements reachable through that strategy's graph, excluding DEACTIVATED and DEPRECATED elements.

**BR-30** Budget propagation uses parent-allocation shares so that a child's total propagated budget across all its parents never exceeds its owned budget.

**BR-31** If an element has multiple active parents in the same strategy, the sum of its allocation percentages across all parents must equal 100%.

**BR-32** An element's consumed budget equals the sum of all its payment records globally across all strategies.

**BR-33** Allocated budget is automatically calculated — it is not set by the user. For each parent, the system sums all outgoing edge contribution percentages and derives each child's allocated budget as a normalized proportional share of the parent's planned budget. If weights do not sum to 100%, they are normalized automatically.

### Progress Rules

**BR-34** Progress can only be logged on elements whose type has the Trackable capability and whose status is ACTIVE.

**BR-35** Every Trackable element maintains two progress values: **Own Progress** (manual entry) and **Calculated Progress** (derived from active children's weighted progress).

**BR-36** For leaf elements (no active children in the strategy), Calculated Progress is not applicable. Only Own Progress is shown.

**BR-37** For parent elements, both values exist. If the difference exceeds the configured discrepancy threshold (default 5%), an alert is shown to prompt the element owner to review.

**BR-38** Draft, Deactivated, and Deprecated children are excluded from parent Calculated Progress.

**BR-39** On Hold and Completed children are included using their frozen last value.

**BR-40** When a child has multiple parents, its progress is first split by its allocation percentage before being weighted among each parent's siblings. This ensures no parent receives more than its share.

---

## 11. VALIDATION RULES

### Graph Integrity

**VR-01** When selecting a parent for an element, the parent picker excludes the element itself and all its existing descendants in the current strategy — cycles cannot be created.

**VR-02** Before creating a new edge, the service verifies no cycle exists. If the proposed parent is already reachable from the proposed child, the request is rejected.

**VR-03** The exact same parent-child pair cannot be created twice in the same strategy.

**VR-04** An element cannot be linked to itself.

**VR-05** An edge cannot connect elements from different organizations.

**VR-06** When creating an edge, the system validates that the parent and child element types conform to the strategy's configured type hierarchy. If not, a warning is shown. The link is created; the warning is displayed visually on the edge.

### Weight Validation

**VR-07** Impact Score input must be a whole number between 1 and 10.

**VR-08** Percentage input must be between 0 and 100.

**VR-09** Allocation percentage must be between 0 and 100. For children with a single active parent, it defaults to 100%.

**VR-10** For any child with multiple active parents, the sum of active incoming allocation percentages must equal 100%.

**VR-11** Weight method must be one of: Percentage, Impact Score, Equal, Business Value, Budget-based.

### Element Type Limits

**VR-12** An organization may not define more than 6 element types.

**VR-13** At least one element type must be defined before a strategy can be created.

**VR-14** An element type cannot be deleted if any element of that type exists (including soft-deleted elements).

**VR-15** An element type's code must be unique within the organization.

### Element Field Rules

**VR-16** Element name is required and must not be empty in at least the default organization language.

**VR-17** End date, if provided, must be on or after start date.

**VR-18** Organization unit, if provided, must reference a valid and active unit.

**VR-19** An element's type cannot be changed after the element is created.

**VR-20** Start date cannot be changed after the element has been activated.

### Status Transition Validation

**VR-21** Any status transition not listed in the allowed map (Section 7.1) is rejected.

**VR-22** A DEPRECATED element cannot have its status changed under any circumstances.

**VR-23** Before activating an element: name required; if Trackable: start date required; if Budgetable: planned budget recommended (warning, not a blocker).

**VR-24** Before completing an element: if Trackable: at least one progress log entry must exist; if Budgetable: alert if consumed < allocated (warning, not a blocker).

**VR-25** Before deprecating an element: element must be in DRAFT, COMPLETED, or DEACTIVATED status. ACTIVE or ON_HOLD elements cannot be deprecated directly.

**VR-26** A strategy cannot transition from DRAFT to ACTIVE without at least one element and a type level configuration.

### Budget Validation

**VR-27** Allocated budget exceeding planned budget shows a warning. Not a hard block.

**VR-28** Payment amount must be positive and non-zero.

**VR-29** Budget and payment fields can only be set on Budgetable elements. Otherwise rejected.

### Progress Validation

**VR-30** Progress rate must be between 0 and 100.

**VR-31** Progress entries can only be created on Trackable Active elements.

**VR-32** Progress entry date must not be in the future.

**VR-33** Milestone date should fall within element's start and end date range. Warning shown if outside (not a hard block).

### Strategy Configuration Validation

**VR-34** Type level references must belong to the same organization as the strategy.

**VR-35** Each selected element type may appear only once in a strategy's type configuration.

**VR-36** Level order values must be unique per strategy.

**VR-37** Strategy effective capability values must be stored per selected type and initialized from the organization-level element type defaults.

**VR-38** A strategy template may not reference deleted element types or types from another organization.

### Indicator & Risk Link Validation

**VR-39** An indicator cannot be linked to the same element more than once in the same strategy.

**VR-40** A risk cannot be linked to the same element more than once in the same strategy.

**VR-41** Indicator and risk links can only be created when the strategy is Draft or Active.

**VR-42** Indicator and risk links cannot be created on elements in COMPLETED, DEACTIVATED, or DEPRECATED status.

---

## 12. CALCULATION PRINCIPLES

This section describes how key metrics are computed. Implementation details are in a separate technical design document.

### Progress

- **Own Progress**: the rate from the most recent manually entered progress log entry. For leaf elements (no active children), this is the only progress value.
- **Calculated Progress**: derived from the weighted average of the element's active children's progress within a given strategy. This is the authoritative value propagated upward to parents.
- Children in DRAFT, DEACTIVATED, or DEPRECATED status are excluded from the calculation. Children in ON_HOLD or COMPLETED contribute their frozen last value.
- When a child has multiple parents in the same strategy, its progress is first split by its allocation percentage before being weighted under each parent. This ensures the child's progress is never counted in full under multiple parents simultaneously.
- If Own Progress and Calculated Progress differ by more than the configured discrepancy threshold (default 5%), the element owner is alerted to review.

### Budget

- **Allocated budget is auto-calculated** — not entered by users.
- For each parent, the system sums all outgoing contribution percentages. Each child's allocated budget equals its proportional share of that parent's planned budget after normalization.
- Consumed budget is the running sum of all payment records on that element, aggregated globally across all strategies.
- When a child has multiple parents, its budget contribution is split by allocation percentage — it is never counted in full under every parent.

### Health Score

- An element's health score is the percentage of its linked KPIs that are meeting their current period targets, weighted by each KPI's contribution weight.
- A health score is only computed when at least one Active KPI is linked.
- Whether a KPI is meeting its target depends on its "bigger-is-better" setting — a KPI is fulfilled if the latest actual meets or exceeds (or is at or below for smaller-is-better) the current period target.

### Risk Score

- An element's risk score is a composite of the urgency levels of its active linked risks.
- The score is weighted 70% on the highest urgency risk and 30% on the average, to ensure critical risks are surfaced prominently.
- Result is expressed as a level: LOW, MEDIUM, HIGH, or CRITICAL.

---

## 13. PERMISSIONS MODEL

### Overview

All authorization in the SPM module is managed centrally. Every permission check is resolved at runtime. User roles are assigned in the application and translate into a set of permissions automatically.

Authorization evaluates two separate concerns:

- **Global element authority**: who owns, edits, shares, or deletes the underlying element record
- **Strategy-scoped usage authority**: who may link, weight, configure, and view that element inside a specific strategy

### Role Definitions

| Role                        | Scope                                 | Description                                                                                     |
| --------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Module Admin**            | SPM module                            | Full administrative access including type configuration and all administrative actions          |
| **Strategy Owner**          | Single strategy                       | Owns and manages a specific strategy                                                            |
| **Strategy Editor**         | Single strategy                       | Can edit strategy configuration and manage element links within it                              |
| **Strategy Viewer**         | Single strategy                       | Read-only access to strategy and its elements                                                   |
| **Element Owner**           | Single element                        | Owns and manages a specific element globally across all strategies                              |
| **Element Editor**          | Single element                        | Can edit a specific element's data globally                                                     |
| **Element Viewer**          | Single element                        | Read-only access to a specific element                                                          |
| **Shared Element Consumer** | Single imported element in a strategy | Can use an imported element in strategy context without gaining ownership of the source element |

### Access Request for Element Types

Users may request access to a specific SPM element type within a scope (a strategy or the entire organization).

**Request flow:**

1. User submits a request specifying target scope (strategy or organization), target element type, and requested role (Viewer or Editor)
2. Request is routed to the Module Admin or the Strategy Owner of the target strategy
3. Approver grants or denies; user is notified
4. If approved, the role is applied for all elements of that type within the requested scope

**Rules:**

- Requesting access to an element type in a strategy grants strategy-scoped rights only — it does not grant global element rights
- Module Admin may grant type-scoped access at the organization level
- All access requests are audited (requestor, approver, granted scope, timestamp)
- Access granted at the element-type scope follows the same top-down inheritance rules as direct element assignments

### Top-Down Permission Inheritance

Permissions on a parent element propagate downward to all its children within the scope of a strategy.

If a user holds strategy-scoped editor permission on a parent element, they inherit strategy-scoped access to all child elements under that parent in the same strategy — and recursively to all descendants.

**Key rules:**

- Inheritance applies within a strategy context only — it does not grant global element rights
- If an element has multiple parents in the same strategy, inherited permissions are the union of all active parent paths
- Removing a parent-child link removes the permissions contributed by that path, unless another active parent path still grants them
- Importing an element never upgrades the importer into a global owner or editor of the source element

### Permission Matrix

| Action                                      | Who Holds This Permission                                                                             |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Browse element library (name, type, status) | Any module user                                                                                       |
| View element full detail                    | Element Viewer, Editor, Owner — OR — inherited from any containing strategy membership                |
| Create a new element                        | Any module user                                                                                       |
| Edit element global fields                  | Element Editor, Element Owner                                                                         |
| Change element status                       | Per transition table in Section 7.1                                                                   |
| Add / remove team member from element       | Element Owner                                                                                         |
| Share element for import by others          | Element Owner, Module Admin                                                                           |
| Import shared element into strategy         | Strategy Editor, Strategy Owner on target strategy — with active share grant from source owner        |
| Link element to a strategy                  | Strategy Editor, Strategy Owner of target strategy — OR — Element Owner of the element being linked   |
| Unlink element from a strategy              | Strategy Editor, Strategy Owner of target strategy — OR — Element Owner of the element being unlinked |
| Update edge weight                          | Strategy Editor, Strategy Owner                                                                       |
| Override strategy-scoped type defaults      | Strategy Editor, Strategy Owner                                                                       |
| Soft-delete an element                      | Element Owner                                                                                         |
| Restore a soft-deleted element              | Module Admin, Strategy Owner of any strategy that contained the element                               |
| Define / modify element types               | Module Admin                                                                                          |
| Create / modify strategy templates          | Module Admin or delegated template manager                                                            |
| Create a strategy                           | Any module user                                                                                       |
| Edit strategy configuration                 | Strategy Owner, Strategy Editor                                                                       |
| Transition strategy status                  | Per transition table in Section 7.3                                                                   |
| Link KPI to element                         | Strategy Editor or Strategy Owner — with strategy-scoped access for that element                      |
| Link risk to element                        | Strategy Editor or Strategy Owner — with strategy-scoped access for that element                      |
| Edit imported element global fields         | Source Element Editor / Element Owner only — not the importing strategy team                          |
| View element metrics                        | Any user with view access to the element                                                              |
| Request access to an element type           | Any module user                                                                                       |
| Approve / deny element type access request  | Module Admin, Strategy Owner (for strategy-scoped requests)                                           |
| Activate all elements in a strategy at once | Strategy Owner, Module Admin                                                                          |
| Activate an individual element              | Element Owner, Element Editor                                                                         |
| Approve strategy status transition          | Per transition table in Section 7.3                                                                   |

---

## 14. NAVIGATION MODEL

### Default Navigation

The default navigation of a strategy follows its type level configuration (e.g., Perspective → Goal → Initiative).

### Alternative Navigation Views

Navigation views are derived automatically from the graph — not stored. Available auto-generated views:

- **Default Map**: top-down following type level order
- **By [Type Name]**: flat list of all elements of a given type (e.g., "All Goals")
- **[Type A] → [Type B]**: two-level collapsed map skipping intermediate types
- **Full Graph**: all nodes and edges as an interactive network

### User Navigation Preference

Each user's preferred view per strategy is persisted individually. When a user opens a strategy, they see their last chosen view. No strategy-level default view is stored — only per-user preferences.

---

## 15. INTEGRATION MAP

### Vision Module

- **Relationship**: Reference only — no structural dependency
- **How it works**: SPM elements with the VisionAligned capability can reference a Vision Priority, Vision Goals, and Vision Indicators from the Vision module. This metadata links the element to national or organizational vision priorities
- **Data flow**: Read from Vision module into SPM element detail. Vision module reports show SPM alignment data as read-only
- **Boundary**: Changes to SPM elements do not affect Vision module data; changes to Vision records are reflected in SPM alignment fields on next load

### Projects Module

- **Relationship**: Reference link — project data remains in the Projects module
- **How it works**: Elements with the Linkable With Project capability can be associated with one or more projects. The link is a reference — no data is copied or synced
- **Data flow**: SPM element detail page shows linked project name and status as read-only. No data flows from SPM into the Projects module
- **Boundary**: Enabling or disabling this capability per element type is controlled within each strategy's type configuration

### Other Modules

No additional hard integrations are planned for Phase 1. Future integration candidates (Finance, HR, Reporting) will be scoped in subsequent design documents.

---

## 16. OPEN QUESTIONS

> This table tracks unresolved design decisions. Each item must be resolved before the related feature can be built or accepted.

| ID    | Question                                                                                                                                                                                | Impact Area              | Owner | Status |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ----- | ------ |
| OQ-01 | Should bulk element activation (US-40) skip non-qualifying elements silently or pause and require user confirmation per failed item?                                                    | Element Lifecycle        | TBD   | Open   |
| OQ-02 | Does access granted through an element-type access request expire after a set period, or is it permanent until manually revoked?                                                        | Permissions              | TBD   | Open   |
| OQ-03 | When a strategy is suspended, should in-progress payments already submitted be allowed to complete, or frozen immediately?                                                              | Budget / Strategy Status | TBD   | Open   |
| OQ-04 | Is there a formal approval gate (separate from activation) required before a strategy can go live — e.g., a sign-off step?                                                              | Strategy Lifecycle       | TBD   | Open   |
| OQ-05 | What notification channels are supported — in-app only, email, or both? Are notifications configurable per user or org-wide?                                                            | Notifications            | TBD   | Open   |
| OQ-06 | When an element is reactivated after a deactivation gap, how is the gap reflected in expected progress calculation? Is the gap excluded from the denominator or treated as 0% progress? | Progress Tracking        | TBD   | Open   |
| OQ-07 | Can a Strategy Editor create new elements from within a strategy, or can they only link existing elements from the library?                                                             | Strategy Building        | TBD   | Open   |
| OQ-08 | Is there a maximum number of elements per strategy, or per organization?                                                                                                                | Limits & Scale           | TBD   | Open   |
| OQ-09 | Should revoked element sharing automatically trigger a review notification to all strategies that have already imported that element?                                                   | Sharing & Permissions    | TBD   | Open   |
| OQ-10 | For the discrepancy threshold (default 5%), is this configurable per strategy, per element type, or only at the organization level?                                                     | Progress Tracking        | TBD   | Open   |

---

## 17. GLOSSARY

| Term                         | Definition                                                                                                                                                                                                     |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SPM Element**              | The single generic building block of a strategy. Any strategic unit — a Perspective, Goal, Initiative, Program, or custom type — is an SPM Element. Its type is defined at the org level.                      |
| **Element Type**             | An organization-level configuration record that gives a name, icon, color, and default capabilities to a category of SPM Elements (e.g., "Goal"). Not hardcoded.                                               |
| **Capability**               | A feature flag on an element type that unlocks additional fields and behaviors — such as budget tracking (Budgetable) or progress logging (Trackable).                                                         |
| **Strategy**                 | A strategic plan that references SPM Elements and defines how they are connected and structured in a specific context. A strategy owns its edges but not its elements.                                         |
| **Strategy Template**        | A reusable configuration blueprint for creating strategies with pre-defined element types, hierarchy rules, and capability defaults. Does not contain live elements.                                           |
| **Edge / Link**              | A directed, weighted connection between two elements within a strategy. Represents that a child element contributes to a parent element. Each edge belongs to one strategy.                                    |
| **Node**                     | An element as it appears within a specific strategy. Each element appears as exactly one node per strategy, even if it has multiple parents.                                                                   |
| **Root Element**             | An element with no parent edges in a given strategy. It sits at the top of the strategy map.                                                                                                                   |
| **Graph**                    | The full set of nodes (elements) and edges (links) within a strategy. The structure is a Directed Acyclic Graph — meaning connections flow one way and no circular paths are allowed.                          |
| **Allocation Percent**       | The portion of a child element's owned value (progress or budget) assigned to a specific parent when that child has multiple parents. Allocations across all parents must sum to 100%.                         |
| **Contribution Weight**      | How strongly a child element counts relative to its siblings under a single parent. Used for local rollup calculation. Independent of allocation percent.                                                      |
| **Own Progress**             | The progress rate manually entered by the element owner or editor via progress log entries. Represents the owner's direct assessment of execution status.                                                      |
| **Calculated Progress**      | The progress rate derived automatically from a parent element's active children's weighted progress. This is the authoritative value used for upward propagation.                                              |
| **Discrepancy Threshold**    | The maximum allowed difference between Own Progress and Calculated Progress before an alert is shown to the element owner. Configurable; default is 5%.                                                        |
| **Health Score**             | A percentage indicating how many of an element's linked KPIs are currently meeting their targets, weighted by each KPI's contribution weight.                                                                  |
| **Risk Score**               | A composite urgency level (LOW / MEDIUM / HIGH / CRITICAL) derived from the urgency ratings of all active linked risks.                                                                                        |
| **Soft Delete**              | A deletion that marks a record as removed without physically removing it from the database. The record is hidden from views and excluded from calculations but can be restored.                                |
| **Orphan Element**           | An element that exists in the organization's element library but is not linked to any strategy. It has no operational impact until linked.                                                                     |
| **Module Admin**             | A user with full administrative access to the SPM module, including type configuration, templates, and override of any element or strategy action.                                                             |
| **Strategy Owner**           | A user who owns a specific strategy. Has full management rights over that strategy including activation, suspension, and completion.                                                                           |
| **Element Owner**            | A user who owns a specific element globally. Has full management rights over that element across all strategies it participates in.                                                                            |
| **Vision Alignment**         | Optional metadata on an SPM Element linking it to a Vision Priority, Vision Goal, or Vision Indicator from the Vision module. Does not create a structural graph dependency.                                   |
| **Type Level Configuration** | The per-strategy configuration that defines which element types participate in the strategy, their allowed parent-child combinations, and the effective capability toggles for each type within that strategy. |

---
