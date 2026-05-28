# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

**Framework**: dbt ≥ 1.9.0 (dbt Cloud, project ID: 275557)  
**Primary Language**: SQL (Jinja-templated), YAML configs  
**Packages**: dbt_utils 1.3.3, dbt-audit-helper  
**Warehouse**: Snowflake (primary), BigQuery, Postgres, Fabric  
**Linting**: SQLFluff (Snowflake dialect, dbt-cloud templater), Ruff (Python)  
**Testing**: Schema tests in YAML, unit tests in YAML, dbt_utils generic tests  
**Semantic Layer**: MetricFlow (semantic models, metrics, saved queries)  
**CI/CD**: GitHub Actions (ci.yml, cd_prod.yml, cd_staging.yml)

## Constitution Check

*GATE: Must pass before implementation. Re-check after design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Layered Modeling | ☐ | Staging models use `{{ source() }}`, marts use `{{ ref() }}` only |
| II. Semantic Layer | ☐ | Mart models exposing quantitative data have semantic model entries |
| III. Testing Discipline | ☐ | PK tests, FK tests, unit tests for non-trivial logic |
| IV. Documentation | ☐ | All models and columns have descriptions, grain stated |
| V. Cost-Aware Dev | ☐ | Using `--select`, no full project runs |

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
models/
├── staging/
│   ├── __sources.yml            # Source definitions (add new sources here)
│   ├── stg_[source_table].sql   # New staging model(s)
│   └── stg_[source_table].yml   # Colocated schema + tests
└── marts/
    ├── [model_name].sql         # New mart model(s)
    └── [model_name].yml         # Colocated schema + tests + unit tests + semantic models

macros/
└── [macro_name].sql             # New macros (if needed)

data-tests/
└── [test_name].sql              # Custom data tests (if needed)
```

## Data Lineage

```text
[Source tables] → [stg_ models] → [mart models] → [semantic layer]
```

| Layer | Models | Materialization | Key Transformations |
|-------|--------|-----------------|---------------------|
| Source | [raw tables from `__sources.yml`] | — | — |
| Staging | [stg_ models] | view | Rename, cast, clean |
| Marts | [mart models] | table | Join, aggregate, business logic |
| Semantic | [semantic models] | — | Entities, dimensions, measures, metrics |

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., intermediate layer] | [current need] | [why direct staging→mart insufficient] |
