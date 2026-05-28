# Tasks: Customer Analytics

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Staging Layer

- [x] T001 Create `models/staging/stg_customers.sql` — CTE with source rename
- [x] T002 Create `models/staging/stg_customers.yml` — schema, descriptions,
  `not_null` + `unique` on `customer_id`

## Phase 2: Mart Model

- [x] T003 Create `models/marts/customers.sql` — join stg_customers with
  orders, compute LTV aggregations and customer_type
- [x] T004 Create `models/marts/customers.yml` — model schema with
  `expression_is_true` (spend consistency), `accepted_values` (customer_type)

## Phase 3: Semantic Layer

- [x] T005 Define semantic model `customers` — primary entity, 4 dimensions,
  4 measures
- [x] T006 Define metric `lifetime_spend_pretax` (simple)
- [x] T007 Define metric `count_lifetime_orders` (simple) +
  `average_order_value` (derived)
- [x] T008 Define saved query `customer_order_metrics` with table export

## Gaps

No gaps identified. Feature is fully tested and documented.
