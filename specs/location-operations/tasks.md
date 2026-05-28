# Tasks: Location Operations

**Status**: migrated — all tasks completed
**Input**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 1: Staging Layer

- [x] T001 Create `models/staging/stg_locations.sql` — CTE, rename,
  truncate opened_at to opened_date
- [x] T002 Create `models/staging/stg_locations.yml` — schema, `not_null` +
  `unique` on `location_id`
- [x] T003 Add unit test `test_does_location_opened_at_trunc_to_date` —
  2 edge cases (midnight, end-of-day)

## Phase 2: Mart + Semantic Layer

- [x] T004 Create `models/marts/locations.sql` — pass-through
- [x] T005 Create `models/marts/locations.yml` — semantic model with
  `location` entity, `location_name` dimension, `opened_date` time
  dimension, `average_tax_rate` measure
- [x] T006 Verify `location` entity is joinable from `orders` semantic
  model via `location_id` FK

## Gaps

- ⚠️ No metrics defined — consider `location_count` or `average_tax_rate`
  as a metric if location-level reporting is needed
- ⚠️ No model-level description in mart `locations.yml`
