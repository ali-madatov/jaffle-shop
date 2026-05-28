# Implementation Plan: Order Intelligence

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Build two fact-level marts (orders, order_items) that enrich raw order data
with product prices, supply costs, and food/drink composition flags. Expose
6 metrics and a daily saved query via the MetricFlow semantic layer.

## Technical Context

**Framework**: dbt >= 1.9.0 (MetricFlow native)
**Materialization**: Staging = view, Mart = table
**Testing**: dbt generic tests, dbt_utils expression tests, native unit tests
**Semantic Layer**: Two semantic models, 6 order-level metrics, 1 saved query

## Implementation Phases

### Phase 1: Staging Layer

1. `stg_orders.sql` — rename columns, derive `subtotal` via
   `order_total - tax_paid`, apply `cents_to_dollars()` macro.
2. `stg_order_items.sql` — rename `id → order_item_id`, `sku → product_id`.
3. YAML schemas with `not_null` + `unique` tests, expression test on
   stg_orders subtotal.

### Phase 2: Mart Models

1. `order_items.sql` — join stg_order_items + stg_orders + stg_products +
   stg_supplies. Aggregate supply_cost per product_id.
2. `orders.sql` — join stg_orders with order_items aggregations. Compute
   `count_food_items`, `count_drink_items`, boolean flags.
3. Schema tests: PK uniqueness, FK relationships, expression consistency.
4. Unit tests: boolean computation, supply cost aggregation.

### Phase 3: Semantic Layer

1. Semantic model `orders`: 3 entities (order_id primary, location + customer
   foreign), 5 dimensions, 4 measures.
2. Semantic model `order_item`: 3 entities, 3 dimensions, 4 measures.
3. 6 metrics: `order_total`, `orders`, `food_orders`, `drink_orders`,
   `new_customer_orders`, `large_orders`.
4. Saved query `order_metrics` — daily time series export.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| order_items grain | One row per order item | Enables per-item profitability |
| Supply cost aggregation | Pre-aggregate per product_id | Avoids fan-out in orders mart |
| Boolean computation | `count > 0` in SQL | Simple, readable, testable |
| Filtered metrics | MetricFlow filter syntax | No custom SQL; reuse `order_count` measure |
| Unit test approach | Mock inputs with minimal columns | Only test relevant logic, not full schema |
| customer_order_number | Window function in stg_orders | Enables new_customer_orders filter |
