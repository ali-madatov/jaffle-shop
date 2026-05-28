# Implementation Plan: Supply Chain & Costs

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Build a supply dimension table that cleans raw supply data (renaming,
composite key generation, currency conversion) and exposes it via MetricFlow
as a dimension-only semantic model. Supply costs feed into the order_items
mart for profitability calculations.

## Technical Context

**Framework**: dbt >= 1.9.0
**Materialization**: Staging = view, Mart = table
**Custom Macro**: `cents_to_dollars()` (dispatch pattern)

## Implementation Phases

### Phase 1: Staging + Mart + Semantic Layer

1. `stg_supplies.sql` — rename columns, generate `supply_uuid` composite
   key from `supply_id || '-' || product_id`, apply `cents_to_dollars()`.
2. `stg_supplies.yml` — schema, PK tests on `supply_uuid`.
3. `supplies.sql` — pass-through from stg_supplies.
4. `supplies.yml` — semantic model with 5 categorical dimensions.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Composite key | `supply_id \|\| '-' \|\| product_id` | Raw data has no single unique key |
| Cost conversion | cents_to_dollars() macro | Consistent with products, adapter-aware |
| Mart logic | Pass-through | No additional transformation needed |
| No metrics | Dimension table only | Costs are aggregated in order_items mart |
