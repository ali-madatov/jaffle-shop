---

description: "Task list template for dbt feature implementation"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Organization**: Tasks follow the dbt modeling layers (sources → staging → marts → semantic → docs/tests).

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Staging models**: `models/staging/stg_[table].sql` + `models/staging/stg_[table].yml`
- **Mart models**: `models/marts/[model].sql` + `models/marts/[model].yml`
- **Sources**: `models/staging/__sources.yml`
- **Macros**: `macros/[macro_name].sql`
- **Custom tests**: `data-tests/[test_name].sql`

<!-- 
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for a dbt feature.
  
  The /speckit.tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Data modeling requirements from spec.md
  - Lineage from plan.md
  
  Tasks follow the dbt modeling flow:
  Sources → Staging → Marts → Semantic Layer → Tests → Docs
  
  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup & Sources

**Purpose**: Source configuration and package dependencies

- [ ] T001 Run `dbt deps` to install/update packages
- [ ] T002 Add new source tables to `models/staging/__sources.yml` with descriptions and `loaded_at_field`
- [ ] T003 [P] Verify source connectivity with `dbt source freshness --select source:[source_name]`

**Checkpoint**: Sources defined and accessible

---

## Phase 2: Staging Models

**Purpose**: Create `stg_` models — one per source table. Views only. Rename, cast, clean.

- [ ] T004 [US1] Create staging SQL in `models/staging/stg_[table].sql` using CTE pattern:
  - Import from `{{ source('ecom', 'raw_[table]') }}`
  - Rename columns, cast types, apply `cents_to_dollars()` where needed
  - Final `select * from renamed`
- [ ] T005 [P] [US1] Create staging YAML in `models/staging/stg_[table].yml`:
  - Model description with grain
  - Column descriptions
  - `unique` + `not_null` on primary key
- [ ] T006 [US1] Validate staging with `dbt build --select stg_[table]`

**Checkpoint**: Staging models build and pass tests — `dbt build --select tag:staging`

---

## Phase 3: Mart Models (User Story 1 - Priority P1) 🎯 MVP

**Goal**: [Brief description of what this mart delivers]

**Independent Test**: `dbt build --select [model_name]`

### Implementation

- [ ] T007 [US1] Create mart SQL in `models/marts/[model].sql`:
  - CTE pattern: import upstream models via `{{ ref() }}`
  - Business logic: joins, aggregations, CASE WHEN
  - Final `select * from [final_cte]`
- [ ] T008 [US1] Create mart YAML in `models/marts/[model].yml`:
  - Model description with grain (e.g., "One row per order")
  - All column descriptions (business meaning, not column name restatement)
  - `unique` + `not_null` on primary key
  - `relationships` on foreign keys → `ref('[upstream_model]')`
  - `dbt_utils.expression_is_true` for cross-column assertions
- [ ] T009 [US1] Add unit tests in `models/marts/[model].yml`:
  - `unit_tests:` block for non-trivial logic (CASE WHEN, aggregations, date math)
  - Define `given:` inputs and `expect:` outputs
- [ ] T010 [US1] Validate with `dbt build --select [model_name]`

**Checkpoint**: User Story 1 mart builds and passes all tests

---

## Phase 4: Mart Models (User Story 2 - Priority P2)

**Goal**: [Brief description of what this mart delivers]

**Independent Test**: `dbt build --select [model_name]`

- [ ] T011 [US2] Create mart SQL in `models/marts/[model].sql`
- [ ] T012 [P] [US2] Create mart YAML in `models/marts/[model].yml` (descriptions, tests)
- [ ] T013 [US2] Add unit tests for non-trivial logic
- [ ] T014 [US2] Validate with `dbt build --select [model_name]`

**Checkpoint**: User Stories 1 AND 2 both pass — `dbt build --select +[model1] +[model2]`

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N-1: Semantic Layer *(if models expose quantitative data)*

**Purpose**: Register models in MetricFlow for the semantic layer

- [ ] TXXX [P] Add `semantic_models:` block in mart YAML:
  - Define `entities:` (primary entity + foreign keys)
  - Define `dimensions:` (categorical + time)
  - Define `measures:` (aggregations)
- [ ] TXXX [P] Define `metrics:` (simple, derived, ratio, or cumulative)
- [ ] TXXX [P] Create `saved_queries:` for common metric combinations
- [ ] TXXX Validate with `dbt sl query --metrics [metric_name] --group-by [dimension]`

---

## Phase N: Polish & Quality

**Purpose**: Linting, documentation completeness, final validation

- [ ] TXXX Run `sqlfluff lint models/` — fix all violations
- [ ] TXXX Run `sqlfluff fix models/` for auto-fixable issues
- [ ] TXXX Verify all models and columns have descriptions
- [ ] TXXX Run full downstream validation: `dbt build --select +[new_models]`
- [ ] TXXX [P] Update `dbt docs generate` and review in dbt docs site
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Staging (Phase 2)**: Depends on sources being defined (Phase 1)
- **Marts (Phase 3+)**: Depend on staging models being built
- **Semantic Layer**: Depends on mart models being built and tested
- **Polish (Final)**: Depends on all models being complete

### dbt DAG Order

```text
Sources → stg_ models (views) → mart models (tables) → semantic layer
```

- Staging models MUST reference sources via `{{ source() }}`
- Mart models MUST reference upstream models via `{{ ref() }}`
- Never skip layers or cross-reference between layers

### Within Each Model

1. Write SQL (CTE pattern)
2. Write YAML (descriptions + schema tests)
3. Add unit tests for non-trivial logic
4. Run `dbt build --select [model]` to validate
5. Run `sqlfluff lint [model_path]` to check style

### Parallel Opportunities

- Staging models for different source tables can be built in parallel [P]
- Mart models with no mutual dependencies can be built in parallel [P]
- YAML schema files can be written in parallel with SQL [P]
- Semantic model definitions can be added in parallel [P]

---

## Build & Validate Commands

```bash
# Build a single model + its tests
dbt build --select [model_name]

# Build model and all upstream dependencies
dbt build --select +[model_name]

# Preview data without materializing
dbt show --select [model_name] --limit 10

# Lint SQL
sqlfluff lint models/[path]/[model].sql

# Run all tests for a model
dbt test --select [model_name]

# Validate semantic layer
dbt sl query --metrics [metric] --group-by [dimension]
```

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story
- Always use `--select` — never run the full project (Constitution V)
- Always use CTE pattern — no subqueries in FROM
- Commit after each model + YAML pair
- Stop at any checkpoint to validate independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
