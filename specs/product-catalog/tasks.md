# Tasks: Product Catalog

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Staging Layer

- [x] T001 Create `models/staging/stg_products.sql` — CTE, rename,
  cents_to_dollars on price
- [x] T002 Create `models/staging/stg_products.yml` — schema, `not_null` +
  `unique` on `product_id`

## Phase 2: Mart + Semantic Layer

- [x] T003 Create `models/marts/products.sql` — pass-through
- [x] T004 Create `models/marts/products.yml` — semantic model with 6
  categorical dimensions
- [x] T005 Verify `products` entity is joinable from `order_item` semantic
  model via `product_id` FK

## Gaps

- ⚠️ No metrics defined on products — consider adding `product_count` or
  `average_product_price` if product-level reporting is needed
- ⚠️ No model-level description in `products.yml` (only semantic model
  has description)
