<!--
Sync Impact Report
==================
Version change: 0.0.0 → 1.0.0 (MAJOR — initial ratification)
Modified principles: N/A (initial creation)
Added sections:
  - Core Principles (5 principles)
  - Technology & Architecture Constraints
  - Development Workflow & Quality Gates
  - Governance
Removed sections: N/A
Templates requiring updates:
  - .specify/templates/plan-template.md ✅ compatible (Constitution Check
    section already references constitution file dynamically)
  - .specify/templates/spec-template.md ✅ compatible (user story format
    aligns with principle III testing requirements)
  - .specify/templates/tasks-template.md ✅ compatible (phase structure
    supports principle-driven task types)
Follow-up TODOs: none
-->

# Jaffle Shop Constitution

## Core Principles

### I. Layered Modeling Architecture

All dbt models MUST follow a strict two-layer architecture:

- **Staging layer** (`models/staging/`): One `stg_` model per source
  table. Materialized as views. Responsible ONLY for renaming, casting,
  and light cleaning. MUST reference sources via `{{ source() }}`.
- **Marts layer** (`models/marts/`): Business-ready models with no
  prefix. Materialized as tables. MUST reference upstream models via
  `{{ ref() }}` — never raw table names or cross-layer source calls.
- New layers (e.g., intermediate) require explicit justification and
  a constitution amendment before adoption.

**Rationale**: Predictable project structure reduces onboarding time
and prevents spaghetti dependencies.

### II. Semantic Layer as Source of Truth

MetricFlow semantic models and metrics MUST be the canonical
definition of business logic:

- Every mart model that exposes quantitative data MUST have a
  corresponding `semantic_models` entry with entities, dimensions,
  and measures.
- Business metrics MUST be defined as dbt metrics (simple, derived,
  ratio, or cumulative) — not as ad-hoc SQL in BI tools.
- Saved queries SHOULD be created for frequently used metric
  combinations.

**Rationale**: A single metric definition prevents conflicting
numbers across dashboards and reports.

### III. Testing Discipline

Every model MUST be tested before merging:

- **Column-level generic tests**: `unique` and `not_null` on primary
  keys; `relationships` on foreign keys; `accepted_values` where
  domain is known.
- **Unit tests**: Required for models containing non-trivial logic
  (CASE WHEN, window functions, date math, regex). Use the native
  dbt unit test format (model/given/expect).
- **Package tests**: Leverage `dbt_utils` for expression-level
  assertions (e.g., `expression_is_true`).
- Tests MUST run via `dbt build --select <model>` (never `dbt run`
  alone) so tests execute alongside the model.

**Rationale**: Catching logic errors before they reach production
protects downstream consumers and the semantic layer.

### IV. Documentation & Discoverability

Every model and every column MUST have a description:

- Model descriptions MUST state the grain (e.g., "One row per
  order").
- Column descriptions MUST explain business meaning, not just
  restate the column name.
- Source YAML (`__sources.yml`) MUST include `loaded_at_field` and
  `freshness` configuration where applicable.
- YAML doc files MUST be colocated with their SQL counterparts
  (same directory).

**Rationale**: Documentation is the interface contract between data
producers and consumers; undocumented models are invisible models.

### V. Cost-Aware Development

All development and CI activity MUST minimize warehouse cost:

- Always use `--select` when running dbt — never execute the full
  project.
- Use `dbt show --limit N` to preview data before materializing.
- Prefer `dbt clone` or deferral (`--defer --state`) over full
  rebuilds in development environments.

---

## Technology & Architecture Constraints

### Stack

- **Framework**: dbt ≥ 1.9.0, dbt Cloud (project ID: 275557)
- **Languages**: SQL (Jinja-templated), YAML configs
- **Packages**: dbt_utils 1.3.3, dbt-audit-helper
- **Warehouses**: Snowflake (primary), BigQuery, Postgres, Fabric
- **Semantic Layer**: MetricFlow (semantic models, metrics, saved queries)

### Project Layout

```text
models/
├── staging/          # stg_ views, __sources.yml
│   ├── stg_*.sql     # One per source table
│   └── stg_*.yml     # Colocated schema YAML
└── marts/            # Business-ready tables
    ├── *.sql
    └── *.yml         # Colocated schema YAML
macros/               # Reusable Jinja macros
seeds/jaffle-data/    # Raw CSV seed data
data-tests/           # Custom data tests
analyses/             # Ad-hoc analysis queries
```

### SQL Style

- **Keywords**: lowercase (`select`, `from`, `where`)
- **Indentation**: 4 spaces
- **Commas**: trailing
- **Pattern**: CTE-based (`with ... as ( ... ), ... select * from ...`)
- **Line length**: 80 characters max
- **Linting**: SQLFluff (Snowflake dialect, dbt-cloud templater)
- **Column grouping**: Use `---------- ids`, `---------- numerics`, etc. comment separators in staging models

### File Naming

- **Convention**: snake_case for all files
- **Staging prefix**: `stg_` for staging models
- **Source config**: `__sources.yml` (double underscore prefix)
- **Schema YAML**: colocated with SQL, same base name (e.g., `orders.sql` + `orders.yml`)

---

## Development Workflow & Quality Gates

### Branching & CI/CD

- **Flow**: WAP (Write-Audit-Publish) — staging → main
- **CI**: GitHub Actions (`ci.yml`) runs on PR
- **CD**: Separate staging (`cd_staging.yml`) and prod (`cd_prod.yml`) deploys
- **Pre-commit hooks**: check-yaml, end-of-file-fixer, trailing-whitespace, ruff

### Build Commands

- **Build model + tests**: `dbt build --select <model>`
- **Preview data**: `dbt show --select <model> --limit 10`
- **Seed data**: `dbt seed --full-refresh --vars '{"load_source_data": true}'`
- **Install packages**: `dbt deps`
- **Lint SQL**: `sqlfluff lint models/`
- **Fix SQL**: `sqlfluff fix models/`
- **Python lint**: `ruff check .`

### Test Requirements (expanded from Principle III)

- **Schema tests**: Defined in colocated `.yml` files
- **Unit tests**: Native dbt format — `unit_tests:` block in model YAML
- **Expression tests**: Use `dbt_utils.expression_is_true` for cross-column assertions
- **Relationship tests**: Required on all foreign key columns

---

## Governance

- **Constitution version**: 1.0.0
- **Amendment process**: Any principle change requires explicit justification
  and review before merging to main
- **Scope**: All dbt models, macros, tests, and semantic definitions in this repository
- Seeds are for reference data only — never use seeds as a
  data-loading mechanism in production.

**Rationale**: Unconstrained warehouse usage accumulates cost that
erodes the value of the data platform.

## Technology & Architecture Constraints

- **dbt version**: >= 1.9.0 (enforced in `dbt_project.yml`).
- **Package dependencies**: Declared in `packages.yml`. Only
  packages from `dbt-labs` or explicitly approved sources.
  Currently: `dbt_utils` (>=1.3.3) and `dbt-audit-helper`.
- **Custom macros**: Use the `dispatch` pattern with adapter-
  specific implementations when SQL differs across warehouses
  (see `macros/cents_to_dollars.sql`).
- **Schema generation**: Non-default environments MUST use the
  custom `generate_schema_name` macro to prefix schemas, ensuring
  isolation between developers and production.
- **Seed data**: Lives in `seeds/jaffle-data/` under the `raw`
  schema. Gated behind `var('load_source_data', false)` to prevent
  accidental execution.

## Development Workflow & Quality Gates

1. **Branch from `v3`** (or the current mainline branch).
2. **Validate locally**: `dbt build --select <changed_models>+`
   with `--warn-error-options '{"error":
   ["NoNodesForSelectionCriteria"]}'`.
3. **Preview data**: `dbt show --select <model> --limit 10` before
   committing materialisation changes.
4. **PR requirements**:
   - All new or modified models MUST have passing tests.
   - All new or modified models MUST have YAML documentation.
   - Semantic layer changes MUST include updated metrics or
     measures where applicable.
5. **CI job**: dbt Cloud CI job MUST pass before merge.

## Governance

This constitution supersedes ad-hoc practices. All pull requests
and code reviews MUST verify compliance with the principles above.

- **Amendments**: Require a PR updating this file, reviewed and
  approved by at least one project maintainer. The amendment MUST
  include a version bump, rationale, and migration plan if
  breaking.
- **Versioning**: MAJOR.MINOR.PATCH semantic versioning.
  MAJOR = principle removal/redefinition; MINOR = new principle or
  material expansion; PATCH = clarification or typo fix.
- **Compliance review**: Quarterly audit of models against
  principles III (testing) and IV (documentation) using
  `dbt-audit-helper`.

**Version**: 1.0.0 | **Ratified**: 2026-05-28 | **Last Amended**: 2026-05-28
