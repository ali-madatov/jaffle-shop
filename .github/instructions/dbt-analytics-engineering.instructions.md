---
applyTo: "models/**/*.sql"
---

# Using dbt for Analytics Engineering

**Core principle:** Apply software engineering discipline (DRY, modularity, testing) to data transformation work through dbt's abstraction layer.

## When to Use

- Building new dbt models, sources, or tests
- Modifying existing model logic or configurations
- Refactoring a dbt project structure
- Creating analytics pipelines or data transformations

## DAG Building Guidelines

- Conform to the existing style: staging (views) → marts (tables)
- Focus heavily on DRY principles
  - Before adding a new model or column, always check if the logic already exists elsewhere
  - Prefer adding a column to an existing model over adding a new model
- **When users request new models:** Always ask "why a new model vs extending existing?" before proceeding

## Model Building Guidelines

- Always use `{{ ref() }}` and `{{ source() }}` over hardcoded table names
- Use CTEs over subqueries
- Before modifying or building on existing models, read their YAML documentation:
  - Find the model's YAML file (colocated with the SQL file)
  - Check the model's `description` to understand its purpose
  - Read column-level `description` fields
  - Review any `meta` properties

## You MUST Look at the Data

When implementing a model, you must use `dbt show` regularly to:
- Preview the input data (relevant columns and values)
- Preview the results of your model
- Run basic data profiling (counts, min, max, nulls)

## Cost Management Best Practices

- Use `--limit` with `dbt show` and insert limits early into CTEs when exploring data
- Use deferral (`--defer --state path/to/prod/artifacts`) to reuse production objects
- Use `dbt clone` for zero-copy clones
- Always use `--select` instead of running the entire project

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| One-shotting models without validation | Iterate with `dbt show` |
| Assuming schema knowledge | Read YAML docs and discover data first |
| Not reading existing model YAML docs | Column names don't reveal business meaning |
| Creating unnecessary models | Extend existing models when possible |
| Hardcoding table names | Always use `{{ ref() }}` and `{{ source() }}` |
| Running DDL directly against warehouse | Use dbt commands exclusively |

**STOP if you're about to:** write SQL without checking column names, modify a model without reading its YAML, skip `dbt show` validation, or create a new model when a column addition would suffice.

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `using-dbt-for-analytics-engineering`
