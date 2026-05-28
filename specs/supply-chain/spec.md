# Feature Specification: Supply Chain & Costs

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Supply Dimension Table (Priority: P1)

As an analyst, I need a clean supply dimension table that maps each supply
item to its product and cost so I can understand the cost structure of each
menu item.

**Acceptance Scenarios**:

1. **Given** a raw supply record, **When** the supplies mart is built,
   **Then** it contains `supply_uuid` (composite key), `supply_id`,
   `product_id`, `supply_name`, `supply_cost`, `is_perishable_supply`.
2. **Given** `supply_cost` in cents, **When** converted, **Then** it is
   in dollars via `cents_to_dollars()` macro.
3. **Given** `supply_uuid`, **When** tested, **Then** `not_null` and
   `unique` pass.

### User Story 2 — Semantic Layer Supply Dimensions (Priority: P2)

As a semantic layer consumer, I need supply dimensions available for
slicing cost metrics by supply name and perishability.

**Acceptance Scenarios**:

1. **Given** the `supplies` semantic model, **When** I query grouped by
   `is_perishable_supply`, **Then** it resolves correctly.

## Requirements

1. `stg_supplies` renames columns, generates `supply_uuid` composite key
   (`supply_id || '-' || product_id`), converts cost via `cents_to_dollars()`.
2. `supplies` mart is a pass-through from `stg_supplies`.
3. PK `supply_uuid` tested with `not_null` + `unique`.
4. Semantic model `supplies` with primary entity `supply`, 5 categorical
   dimensions.
5. No metrics defined (dimension table feeding cost aggregations in
   order_items).

## Dependencies

- **Upstream**: `source('ecom', 'raw_supplies')`
- **Downstream**: `ref('order_items')` mart aggregates `supply_cost` per
  product

## Files

| File | Purpose |
|------|---------|
| `models/staging/stg_supplies.sql` | Staging: rename, composite key, cents_to_dollars |
| `models/staging/stg_supplies.yml` | Schema + PK tests |
| `models/marts/supplies.sql` | Mart: pass-through |
| `models/marts/supplies.yml` | Semantic model (dimensions only) |
