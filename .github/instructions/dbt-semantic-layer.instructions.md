---
applyTo: "models/**/*.yml"
---

# Building the dbt Semantic Layer

This skill guides creation and modification of dbt Semantic Layer components: semantic models, entities, dimensions, metrics, and saved queries.

## Determine Which Spec to Use

This project uses the **legacy spec** (dbt Core 1.9+):
- Semantic models are defined as separate top-level `semantic_models:` resources
- Metrics are defined as top-level `metrics:` resources

## Entry Points

### Business Question First
1. Search project models for relevant candidates
2. Present top matches with context
3. User confirms which model(s) to build on
4. Work backwards from user's need to define entities, dimensions, and metrics

### Model First
1. Read the model SQL and existing YAML config
2. Identify the grain (primary key / entity)
3. Suggest dimensions based on column types
4. Ask what metrics the user wants to define

## Legacy Spec Structure (This Project)

```yaml
semantic_models:
  - name: orders
    model: ref('orders')
    defaults:
      agg_time_dimension: ordered_at
    entities:
      - name: order_id
        type: primary
      - name: customer
        type: foreign
        expr: customer_id
    dimensions:
      - name: ordered_at
        type: time
        type_params:
          time_granularity: day
      - name: is_food_order
        type: categorical
    measures:
      - name: order_total
        agg: sum
      - name: order_count
        expr: 1
        agg: sum

metrics:
  - name: order_total
    type: simple
    label: Order Total
    type_params:
      measure: order_total
```

## Metric Types

| Type | Use Case | Example |
|------|----------|---------|
| **Simple** | Direct aggregation of a measure | `order_total` (sum of amounts) |
| **Derived** | Combine metrics with math | `average_order_value` (LTV / orders) |
| **Cumulative** | Running totals, trailing windows | 7-day rolling revenue |
| **Ratio** | Numerator / denominator | `food_revenue_pct` |
| **Conversion** | Funnel analysis | Visit → purchase rate |

## Filtering Metrics

```yaml
filter: |
  {{ Dimension('order_id__is_food_order') }} = true
```

**Important**: Filter expressions can only reference columns declared as dimensions or entities.

## Validation

1. **Parse**: `dbt parse` to confirm YAML syntax
2. **Semantic Layer**: `dbt sl validate` or `mf validate-configs`

Do not consider work complete until both validations pass.

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Missing time dimension | Every semantic model with measures needs a default time dimension |
| Using `window` and `grain_to_date` together | Cumulative metrics can only have one |
| Filtering on non-dimension columns | Only declared dimensions/entities can be used in filters |
| `mf validate-configs` shows stale results | Re-run `dbt parse` first |

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `building-dbt-semantic-layer`
