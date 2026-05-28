# Implementation Plan: Product Catalog

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Build a product dimension table that cleans raw product data and exposes it
via the MetricFlow semantic layer as a dimension-only semantic model.

## Technical Context

**Framework**: dbt >= 1.9.0
**Materialization**: Staging = view, Mart = table
**Custom Macro**: `cents_to_dollars()` (dispatch pattern, multi-adapter)

## Implementation Phases

### Phase 1: Staging + Mart + Semantic Layer

1. `stg_products.sql` — rename columns, apply `cents_to_dollars()` to price.
2. `stg_products.yml` — schema, PK tests.
3. `products.sql` — pass-through from stg_products.
4. `products.yml` — semantic model with 6 categorical dimensions.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Mart logic | Pass-through | No additional business logic needed |
| Price conversion | cents_to_dollars() macro | Reusable, adapter-aware |
| Semantic model | Dimensions only, no measures | Product is a dimension table |
