# Tasks: Revenue & Profitability

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Core Measures & Simple Metrics

- [x] T001 Define measures on `order_item` semantic model: revenue,
  food_revenue, drink_revenue, median_revenue
- [x] T002 Define simple metric `revenue` (sum of product_price)
- [x] T003 Define simple metrics `food_revenue`, `drink_revenue`,
  `median_revenue`, `order_cost`

## Phase 2: Advanced Metrics

- [x] T004 Define ratio metrics `food_revenue_pct`, `drink_revenue_pct`
- [x] T005 Define derived metric `revenue_growth_mom` with 1-month offset
- [x] T006 Define derived metric `order_gross_profit` (revenue - cost)
- [x] T007 Define cumulative metric `cumulative_revenue`

## Phase 3: Export

- [x] T008 Define saved query `revenue_metrics` — daily grain, table export

## Gaps

No gaps identified. All 5 metric types (simple, ratio, derived, cumulative)
are represented with correct MetricFlow syntax.
