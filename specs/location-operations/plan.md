# Implementation Plan: Location Operations

**Branch**: `migrated` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)
**Status**: migrated — reverse-engineered from existing implementation

## Summary

Build a location dimension table that cleans raw store data (renaming,
date truncation) and exposes it via MetricFlow with an `average_tax_rate`
measure. Includes a unit test for timestamp-to-date truncation.

## Technical Context

**Framework**: dbt >= 1.9.0
**Materialization**: Staging = view, Mart = table
**Testing**: Generic tests + 1 unit test for date truncation

## Implementation Phases

### Phase 1: Staging + Mart + Semantic Layer

1. `stg_locations.sql` — rename columns, truncate `opened_at` timestamp
   to `opened_date` using date cast.
2. `stg_locations.yml` — schema, PK tests, unit test with 2 edge cases
   (midnight and end-of-day timestamps).
3. `locations.sql` — pass-through from stg_locations.
4. `locations.yml` — semantic model with `location` primary entity,
   2 dimensions, 1 measure.

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Date truncation | Cast in SQL | Simple, no macro needed |
| Unit test placement | In stg_locations.yml | Tests the staging transformation, not the mart |
| Semantic model | 1 measure (average_tax_rate) | Tax rate is the only numeric attribute |
