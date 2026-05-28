# Feature Specification: Revenue & Profitability

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Revenue Analytics (Priority: P1)

As a finance analyst, I need revenue metrics broken down by food vs. drink
categories so I can track product mix trends over time.

**Acceptance Scenarios**:

1. **Given** the `revenue` metric, **When** queried by day, **Then** it
   returns `sum(product_price)` across all order items.
2. **Given** `food_revenue` and `drink_revenue`, **When** summed, **Then**
   they account for all revenue (food items + drink items).
3. **Given** `food_revenue_pct`, **When** queried, **Then** it equals
   `food_revenue / revenue`.

### User Story 2 — Profitability Metrics (Priority: P1)

As a finance analyst, I need gross profit metrics that subtract supply
costs from revenue so I can assess margin health.

**Acceptance Scenarios**:

1. **Given** the `order_gross_profit` metric, **When** queried, **Then**
   it equals `revenue - order_cost`.
2. **Given** the `order_cost` metric, **When** queried, **Then** it
   returns the sum of supply costs per order.

### User Story 3 — Growth & Cumulative Tracking (Priority: P2)

As a leadership stakeholder, I need month-over-month revenue growth and
cumulative revenue metrics for trend analysis.

**Acceptance Scenarios**:

1. **Given** `revenue_growth_mom`, **When** queried, **Then** it computes
   `(current_revenue - revenue_prev_month) * 100 / revenue_prev_month`.
2. **Given** `cumulative_revenue`, **When** queried, **Then** it returns
   a running total of all revenue.

## Requirements

1. All revenue metrics are sourced from the `order_item` semantic model
   (defined in `order_items.yml`).
2. 5 simple metrics: `revenue`, `order_cost`, `median_revenue`,
   `food_revenue`, `drink_revenue`.
3. 2 ratio metrics: `food_revenue_pct`, `drink_revenue_pct`.
4. 2 derived metrics: `revenue_growth_mom` (offset window),
   `order_gross_profit`.
5. 1 cumulative metric: `cumulative_revenue`.
6. Saved query `revenue_metrics` — daily grain, food + drink revenue.
7. Unit test `test_supply_costs_sum_correctly` validates cost aggregation
   (shared with order-intelligence).

## Dependencies

- **Upstream**: `order_items` mart (semantic model `order_item`)
- **Shared**: `order_cost` metric defined in `orders.yml`, referenced
  by `order_gross_profit`

## Files

| File | Purpose |
|------|---------|
| `models/marts/order_items.yml` | Semantic model `order_item`, measures, 10 metrics, saved query |
