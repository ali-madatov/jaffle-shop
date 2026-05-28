# Tasks: Order Intelligence

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Staging Layer

- [x] T001 Create `models/staging/stg_orders.sql` — CTE, rename, derive subtotal
- [x] T002 Create `models/staging/stg_orders.yml` — schema, expression test
- [x] T003 Create `models/staging/stg_order_items.sql` — CTE, rename, remap SKU
- [x] T004 Create `models/staging/stg_order_items.yml` — schema, PK tests

## Phase 2: Mart Models

- [x] T005 Create `models/marts/order_items.sql` — join 4 staging models,
  aggregate supply_cost
- [x] T006 Create `models/marts/order_items.yml` — schema, FK test, unit test
  `test_supply_costs_sum_correctly`
- [x] T007 Create `models/marts/orders.sql` — join stg_orders with order_items
  aggregations, compute booleans
- [x] T008 Create `models/marts/orders.yml` — schema, 2 expression tests, FK test,
  unit test `test_order_items_compute_to_bools_correctly`

## Phase 3: Semantic Layer

- [x] T009 Define semantic model `orders` — 3 entities, 5 dimensions, 4 measures
- [x] T010 Define semantic model `order_item` — 3 entities, 3 dimensions, 4 measures
- [x] T011 Define 6 metrics: order_total, orders, food_orders, drink_orders,
  new_customer_orders, large_orders
- [x] T012 Define saved query `order_metrics` — daily export as table

## Gaps

- ⚠️ No unit test for `customer_order_number` window function logic
  (consider adding via `/speckit.specify`)
