---
applyTo: "models/**/*.yml"
---

# Adding dbt Unit Tests

## What Are Unit Tests in dbt

dbt unit tests validate SQL modeling logic on static inputs before materializing in production. If any unit test for a model fails, dbt will not materialize that model.

## When to Use

Unit test a model when:
- SQL contains complex logic: regex, date math, window functions, multi-condition `case when`, truncation, complex joins
- Writing custom logic to process input data
- Logic had bugs reported before
- Edge cases not yet seen in actual data
- Prior to refactoring transformation logic
- Models with high criticality (public, contracted, or upstream of an exposure)

**When NOT to use:** Don't test built-in SQL functions (`min()`, `sum()`, etc.) — they're tested by the warehouse provider.

## General Format (Model-Inputs-Outputs)

```yaml
unit_tests:
  - name: test_descriptive_name
    description: >
      Scenario: [describe the scenario]
    model: model_name
    given:
      - input: ref('upstream_model')
        rows:
          - {col1: value1, col2: value2}
      - input: ref('other_model')
        rows:
          - {col1: value1}
    expect:
      rows:
        - {expected_col1: value1, expected_col2: value2}
```

## Key Rules

1. **Use `dict` format first** (default, most readable). Fall back to `csv` or `sql` only when needed
2. **Only include relevant columns** in mock data — don't mock all columns
3. **Run with `dbt build --select model_name`** (recommended) — runs unit tests + builds + data tests
4. **Exclude from production:** `--exclude-resource-type unit_test` or `DBT_EXCLUDE_RESOURCE_TYPES`
5. **If model depends on ephemeral model**, you MUST use `format: sql` for that input
6. Unit tests must be defined in a YAML file in your `model-paths` directory

## Executing Unit Tests

```bash
# Run unit tests + build + data tests
dbt build --select model_name

# Only run unit tests
dbt test --select "model_name,test_type:unit"

# Specific unit test
dbt test --select test_name
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Testing simple SQL built-in functions | Only unit test complex logic |
| Mocking all columns in input data | Only include columns relevant to the test |
| Using `sql` format when `dict` works | Prefer `dict`, fall back to `csv`/`sql` when needed |
| Missing `input` for a `ref`/`source` | Include all dependencies to avoid "node not found" errors |
| Testing Python models or snapshots | Unit tests only support SQL models |

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `adding-dbt-unit-test`
