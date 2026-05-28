# Feature Specification: Location Operations

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Location Dimension Table (Priority: P1)

As an analyst, I need a clean location dimension table with standardized
names, tax rates, and opening dates so I can analyze performance by
store location.

**Acceptance Scenarios**:

1. **Given** a raw store record with `opened_at` as a timestamp, **When**
   the stg_locations model is built, **Then** `opened_date` is truncated
   to a date (no time component).
2. **Given** `location_id`, **When** tested, **Then** `not_null` and
   `unique` pass.
3. **Given** `opened_at = '2016-09-01T00:00:00'`, **When** truncated,
   **Then** `opened_date = '2016-09-01'`.

### User Story 2 — Semantic Layer Location Dimensions (Priority: P2)

As a semantic layer consumer, I need location dimensions and an
`average_tax_rate` measure for slicing order metrics by store.

**Acceptance Scenarios**:

1. **Given** the `locations` semantic model, **When** I query `order_total`
   grouped by `location_name`, **Then** it resolves via the `location`
   entity FK on the orders semantic model.
2. **Given** the `average_tax_rate` measure, **When** queried, **Then**
   it returns the average `tax_rate` across locations.

## Requirements

1. `stg_locations` renames `id → location_id`, `name → location_name`,
   `opened_at → opened_date` (truncated to date).
2. `locations` mart is a pass-through from `stg_locations`.
3. PK `location_id` tested with `not_null` + `unique`.
4. Unit test `test_does_location_opened_at_trunc_to_date` validates
   timestamp truncation on two edge cases.
5. Semantic model `locations` with primary entity `location`,
   `location_name` (categorical), `opened_date` (time, day granularity),
   and `average_tax_rate` measure.

## Dependencies

- **Upstream**: `source('ecom', 'raw_stores')`
- **Downstream**: `ref('orders')` mart references `location_id` as FK

## Files

| File | Purpose |
|------|---------|
| `models/staging/stg_locations.sql` | Staging: rename, date truncation |
| `models/staging/stg_locations.yml` | Schema, PK tests, unit test |
| `models/marts/locations.sql` | Mart: pass-through |
| `models/marts/locations.yml` | Semantic model with measure |
