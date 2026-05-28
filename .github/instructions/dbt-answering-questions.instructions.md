---
applyTo: "**"
---

# Answering Natural Language Questions with dbt

## Overview

Answer data questions using the best available method: semantic layer first, then SQL modification, then model discovery, then manifest analysis. Always exhaust options before saying "cannot answer."

**Use for:** Business questions that need data answers
- "What were total sales last month?"
- "How many active customers do we have?"
- "Show me revenue by region"

**Not for:** Building/modifying dbt models, running tests, or `dbt build` workflows.

## Decision Flow (Priority Order)

| Priority | Condition | Approach | Tools |
|----------|-----------|----------|-------|
| 1 | Semantic layer active | Query metrics directly | `list_metrics`, `get_dimensions`, `query_metrics` |
| 2 | SL active but needs modifications | Modify compiled SQL | `get_metrics_compiled_sql`, then `execute_sql` |
| 3 | No SL, discovery tools active | Explore models, write SQL | `get_mart_models`, `get_model_details`, `dbt show` |
| 4 | No MCP, in dbt project | Analyze artifacts, write SQL | Read `target/manifest.json`, `target/catalog.json` |

## This Project's Semantic Layer

Available metrics (query these first):
- **Orders**: `order_total`, `orders`, `food_orders`, `drink_orders`, `new_customer_orders`, `large_orders`
- **Revenue**: `revenue`, `food_revenue`, `drink_revenue`, `food_revenue_pct`, `drink_revenue_pct`, `revenue_growth_mom`, `order_gross_profit`, `cumulative_revenue`, `median_revenue`, `order_cost`
- **Customers**: `lifetime_spend_pretax`, `count_lifetime_orders`, `average_order_value`

Saved queries: `order_metrics`, `revenue_metrics`, `customer_order_metrics`

## Suggesting Improvements

| Gap | Suggestion |
|-----|------------|
| Metric doesn't exist | "Add a metric definition to your semantic model" |
| Dimension missing | "Add dimension to the semantic model's dimensions list" |
| No semantic layer | "Consider adding a semantic layer for this data" |

**Stay at semantic layer level.** Do NOT suggest database schema changes or ETL pipeline modifications.

## Red Flags — STOP

- Writing SQL without checking if semantic layer can answer
- Saying "cannot answer" without trying all 4 approaches
- Using staging models when mart models exist
- Reading entire manifest.json without filtering

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `answering-natural-language-questions-with-dbt`
