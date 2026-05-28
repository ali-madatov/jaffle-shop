# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`  
**Created**: [DATE]  
**Status**: Draft  
**Input**: User description: "$ARGUMENTS"

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  For dbt features, each story typically maps to a model or set of models that can be
  built and tested independently with `dbt build --select <model>`.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
-->

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [e.g., "Can be tested with `dbt build --select <model>` and delivers [specific value]"]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 - [Brief Title] (Priority: P2)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

[Add more user stories as needed, each with an assigned priority]

### Edge Cases

- What happens when [source data is missing or NULL]?
- How does model handle [late-arriving facts]?
- What if [upstream source schema changes]?

## Data Modeling Requirements *(mandatory)*

### Sources & Staging

- **Source(s)**: [List raw tables from `__sources.yml` or new sources needed]
- **Staging models**: [List `stg_` models needed — one per source table]
- **Transformations**: [Renaming, casting, cleaning applied in staging]

### Mart Models

- **Model name(s)**: [e.g., `orders`, `customers`]
- **Grain**: [e.g., "One row per order", "One row per customer"]
- **Materialization**: [table / incremental / view]
- **Key columns**: [Primary key, foreign keys, important dimensions/measures]
- **Business logic**: [Aggregations, CASE WHEN logic, window functions, date math]

### Semantic Layer *(include if model exposes quantitative data)*

- **Semantic model**: [Entity name, primary entity, dimensions, measures]
- **Metrics**: [Metric definitions — simple, derived, ratio, or cumulative]
- **Saved queries**: [Frequently used metric combinations]

## Testing Requirements *(mandatory)*

### Schema Tests (in colocated `.yml`)

- **Primary key**: `unique` + `not_null` on [column]
- **Foreign keys**: `relationships` on [column] → `ref('[upstream_model]')`
- **Accepted values**: [column] must be one of [values]
- **Expression tests**: `dbt_utils.expression_is_true` for [cross-column assertion]

### Unit Tests (in model `.yml`)

- **Test name**: [e.g., `test_order_total_calculation`]
- **Model**: [target model]
- **Given/Expect**: [Input rows and expected output for non-trivial logic]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Model MUST [specific transformation, e.g., "aggregate orders by customer"]
- **FR-002**: Model MUST [data quality rule, e.g., "exclude soft-deleted records"]
- **FR-003**: Model MUST [business logic, e.g., "classify customers as new vs returning"]

### Upstream Dependencies

- **Models**: [List `{{ ref() }}` dependencies]
- **Sources**: [List `{{ source() }}` dependencies]
- **Macros**: [List any custom macros used, e.g., `cents_to_dollars`]
- **Packages**: [dbt_utils tests or macros needed]

## Documentation Requirements *(mandatory)*

- **Model description**: [Grain statement — "One row per [entity]"]
- **Column descriptions**: [Business meaning for each column, not just column name]

## Success Criteria *(mandatory)*

- **SC-001**: [e.g., "`dbt build --select <model>` passes all tests"]
- **SC-002**: [e.g., "All columns have descriptions in YAML"]
- **SC-003**: [e.g., "Semantic model registered and queryable in MetricFlow"]
- **SC-004**: [e.g., "SQLFluff lint passes with zero violations"]

## Assumptions

- [e.g., "Source data is loaded via seeds or EL tool before model runs"]
- [e.g., "Staging model for this source already exists"]
- [e.g., "No new dbt packages required"]
