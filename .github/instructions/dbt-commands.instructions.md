---
applyTo: "**"
---

# Running dbt Commands

## Preferences

1. **Always use `build` — even when users say "run"** — `build` = `run` + `test` in one step
2. **Always use `--quiet`** with `--warn-error-options '{"error": ["NoNodesForSelectionCriteria"]}'`
3. **Always use `--select`** — never run the entire project without explicit approval

## Quick Reference

```bash
# Standard command pattern
dbt build --select my_model --quiet --warn-error-options '{"error": ["NoNodesForSelectionCriteria"]}'

# Preview model output
dbt show --select my_model --limit 10

# Run inline SQL query
dbt show --inline "select * from {{ ref('orders') }}" --limit 5

# With variables
dbt build --select my_model --vars '{"key": "value"}'

# Full refresh for incremental models
dbt build --select my_model --full-refresh

# List resources before running
dbt list --select my_model+ --resource-type model
```

## Selectors

**Always provide a selector.** Graph operators:

| Operator | Meaning | Example |
|----------|---------|---------|
| `model+` | Model + all downstream | `stg_orders+` |
| `+model` | Model + all upstream | `+customers` |
| `+model+` | Both directions | `+orders+` |
| `model+N` | Model + N levels downstream | `stg_orders+1` |

```bash
--select my_model              # Single model
--select staging.*             # Path pattern
--select model_a model_b       # Union (space)
--select tag:x,config.mat:y    # Intersection (comma)
--exclude my_model             # Exclude from selection
```

## Show

Preview data with `dbt show`. Use `--inline` for arbitrary SQL queries.

**Important:** Use `--limit` flag, not SQL `LIMIT` clause.

## Analyzing Run Results

```bash
cat target/run_results.json | jq '.results[] | {node: .unique_id, status: .status, time: .execution_time}'
cat target/run_results.json | jq '.results[] | select(.status != "success")'
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `test` after model change | Use `build` — test doesn't refresh the model |
| Running without `--select` | Always specify what to run |
| Using `--quiet` without warn-error | Add `--warn-error-options` |
| Adding LIMIT to SQL in `dbt show` | Use `--limit` parameter instead |
| Vars with special characters | Pass as simple string, no `\` or `\n` |

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `running-dbt-commands`
