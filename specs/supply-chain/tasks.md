# Tasks: Supply Chain & Costs

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Staging Layer

- [x] T001 Create `models/staging/stg_supplies.sql` — CTE, rename,
  composite key generation, cents_to_dollars
- [x] T002 Create `models/staging/stg_supplies.yml` — schema, `not_null` +
  `unique` on `supply_uuid`

## Phase 2: Mart + Semantic Layer

- [x] T003 Create `models/marts/supplies.sql` — pass-through
- [x] T004 Create `models/marts/supplies.yml` — semantic model with
  `supply` primary entity, 5 categorical dimensions
- [x] T005 Verify supply_cost flows correctly into `order_items` mart
  via `stg_supplies` reference

## Gaps

- ⚠️ No metrics defined on supplies — consider `total_supply_cost` or
  `perishable_supply_pct` if supply-level reporting is needed
- ⚠️ No model-level description in mart `supplies.yml`
