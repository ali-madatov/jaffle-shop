# Implementation Plan: Revenue & Profitability

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Define 10 MetricFlow metrics on the `order_item` semantic model covering
revenue breakdown (food/drink), profitability (gross profit), growth tracking
(MoM), and cumulative revenue. Includes a daily saved query export.

## Technical Context

**Framework**: dbt >= 1.9.0 (MetricFlow native)
**Semantic Model**: `order_item` (defined in order_items.yml)
**Metric Types Used**: simple, ratio, derived (with offset_window), cumulative

## Implementation Phases

### Phase 1: Core Revenue Measures & Metrics

1. Define measures on `order_item` semantic model: `revenue`
   (`sum(product_price)`), `food_revenue`, `drink_revenue`,
   `median_revenue`.
2. Define 5 simple metrics: `revenue`, `order_cost`, `median_revenue`,
   `food_revenue`, `drink_revenue`.

### Phase 2: Advanced Metrics & Export

1. Define 2 ratio metrics: `food_revenue_pct`, `drink_revenue_pct`.
2. Define `revenue_growth_mom` — derived metric with 1-month offset window.
3. Define `order_gross_profit` — derived metric: `revenue - order_cost`.
4. Define `cumulative_revenue` — cumulative metric on revenue measure.
5. Define `revenue_metrics` saved query — daily grain export.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Revenue source | product_price, not order_total | Per-item granularity for mix analysis |
| Food/drink measures | CASE WHEN in measure expr | MetricFlow handles conditional aggregation |
| MoM growth | offset_window: 1 month | Native MetricFlow feature, no custom SQL |
| Gross profit | Derived from 2 metrics | Ensures consistency with individual metrics |
| Cumulative metric | All-time (no window) | Business wants running total |
