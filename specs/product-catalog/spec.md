# Feature Specification: Product Catalog

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Product Dimension Table (Priority: P1)

As an analyst, I need a clean product dimension table with standardized
names and food/drink classification so I can join products to order items
and filter by product type.

**Acceptance Scenarios**:

1. **Given** a raw product record, **When** the products mart is built,
   **Then** it contains `product_id`, `product_name`, `product_type`,
   `product_description`, `product_price`, `is_food_item`, `is_drink_item`.
2. **Given** `product_price`, **When** sourced from raw, **Then** it has
   been converted from cents to dollars via `cents_to_dollars()` macro.
3. **Given** `product_id`, **When** tested, **Then** `not_null` and
   `unique` pass.

### User Story 2 — Semantic Layer Dimensions (Priority: P2)

As a semantic layer consumer, I need product dimensions available for
slicing metrics by product name, type, and food/drink classification.

**Acceptance Scenarios**:

1. **Given** the `products` semantic model, **When** I query a metric
   grouped by `product_name`, **Then** it resolves correctly via the
   `product` entity.

## Requirements

1. `stg_products` renames columns, converts `price` from cents to dollars.
2. `products` mart is a pass-through from `stg_products` (no additional logic).
3. PK `product_id` tested with `not_null` + `unique`.
4. Semantic model `products` with primary entity `product`, 6 categorical
   dimensions.
5. No metrics defined (pure dimension table).

## Dependencies

- **Upstream**: `source('ecom', 'raw_products')`
- **Downstream**: `ref('order_items')` mart, `ref('supplies')` mart

## Files

| File | Purpose |
|------|---------|
| `models/staging/stg_products.sql` | Staging: rename, cents_to_dollars |
| `models/staging/stg_products.yml` | Schema + PK tests |
| `models/marts/products.sql` | Mart: pass-through |
| `models/marts/products.yml` | Semantic model (dimensions only) |
