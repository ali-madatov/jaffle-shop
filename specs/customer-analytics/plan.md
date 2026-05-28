# Implementation Plan: Customer Analytics

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Build a customer-grain data mart that aggregates order history into lifetime
metrics (spend, order count, customer type) and exposes them via MetricFlow
semantic layer with three metrics and a saved query.

## Technical Context

**Framework**: dbt >= 1.9.0 (MetricFlow native)
**Materialization**: Staging = view, Mart = table
**Testing**: dbt generic tests + expression tests via dbt_utils
**Semantic Layer**: MetricFlow semantic models, metrics, saved queries

## Implementation Phases

### Phase 1: Staging Layer

1. Create `stg_customers.sql` — CTE pattern, rename `id → customer_id`,
   `name → customer_name` from `source('ecom', 'raw_customers')`.
2. Create `stg_customers.yml` — document model + columns, add `not_null` +
   `unique` on `customer_id`.

### Phase 2: Mart Layer + Semantic Layer

1. Create `customers.sql` — join `stg_customers` with order aggregations
   from `ref('orders')`. Compute `count_lifetime_orders`,
   `first_ordered_at`, `last_ordered_at`, `lifetime_spend_pretax`,
   `lifetime_tax_paid`, `lifetime_spend`, `customer_type`.
2. Create `customers.yml`:
   - Model schema with `expression_is_true` test on spend consistency.
   - `not_null` + `unique` on `customer_id`.
   - `accepted_values` on `customer_type`.
   - Semantic model: primary entity `customer`, 4 dimensions, 4 measures.
   - 3 metrics: `lifetime_spend_pretax`, `count_lifetime_orders`,
     `average_order_value` (derived).
   - Saved query: `customer_order_metrics` exported as table.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Join target for orders | `ref('orders')` mart, not staging | Reuse enriched order data including subtotals |
| customer_type logic | CASE WHEN in SQL | Simple boolean, no need for macro |
| LTV metric type | Derived (`pretax / orders`) | MetricFlow handles the ratio correctly |
| Saved query grouping | By `Entity('customer')` | One row per customer in export |
| Materialization | Table | Downstream BI queries need fast scans |
