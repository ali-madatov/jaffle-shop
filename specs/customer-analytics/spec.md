# Feature Specification: Customer Analytics

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Customer Lifetime Value Reporting (Priority: P1)

As a business analyst, I need a consolidated customer mart that aggregates
each customer's order history into lifetime metrics so that I can segment
customers by value and recency.

**Acceptance Scenarios**:

1. **Given** a customer with 3 orders totaling $90 pre-tax, **When** the
   customers mart is built, **Then** `count_lifetime_orders = 3` and
   `lifetime_spend_pretax = 90`.
2. **Given** a customer whose `lifetime_spend_pretax + lifetime_tax_paid`,
   **When** compared to `lifetime_spend`, **Then** they are equal
   (expression test).
3. **Given** a customer with exactly one order, **When** `customer_type` is
   computed, **Then** the value is `'new'`.
4. **Given** a customer with more than one order, **When** `customer_type`
   is computed, **Then** the value is `'returning'`.

### User Story 2 — Semantic Layer Metrics for Customers (Priority: P1)

As a data consumer querying via the semantic layer, I need `lifetime_spend_pretax`,
`count_lifetime_orders`, and `average_order_value` metrics defined on the
customers semantic model so that I can answer business questions consistently.

**Acceptance Scenarios**:

1. **Given** the customers semantic model, **When** I query
   `lifetime_spend_pretax` grouped by `customer_type`, **Then** I get
   accurate totals for new vs. returning customers.
2. **Given** the derived metric `average_order_value`, **When** I query it,
   **Then** it equals `lifetime_spend_pretax / count_lifetime_orders`.

### User Story 3 — Saved Query Export (Priority: P2)

As a BI tool consumer, I need a pre-built `customer_order_metrics` saved
query exported as a table so that downstream dashboards have a stable,
optimized data source.

**Acceptance Scenarios**:

1. **Given** the saved query `customer_order_metrics`, **When** exported,
   **Then** it materializes a table grouped by customer entity with
   `count_lifetime_orders`, `lifetime_spend_pretax`, and
   `average_order_value`.

## Requirements

1. Staging model `stg_customers` renames `id → customer_id` and
   `name → customer_name` from `source('ecom', 'raw_customers')`.
2. Mart model `customers` joins `stg_customers` with order aggregations
   from the `orders` mart (not staging).
3. `customer_type` is derived: `'returning'` if `count > 1`, else `'new'`.
4. `lifetime_spend = lifetime_spend_pretax + lifetime_tax_paid` (tested).
5. Primary key `customer_id` has `not_null` + `unique` tests.
6. `customer_type` has `accepted_values` test for `['new', 'returning']`.
7. Semantic model declares `customer` as primary entity on `customer_id`.
8. Three metrics: `lifetime_spend_pretax` (simple), `count_lifetime_orders`
   (simple), `average_order_value` (derived).
9. Saved query `customer_order_metrics` exports as table.

## Dependencies

- **Upstream**: `source('ecom', 'raw_customers')`, `ref('orders')` mart
- **Downstream**: Referenced as FK target from `orders` mart

## Files

| File | Purpose |
|------|---------|
| `models/staging/stg_customers.sql` | Staging: rename + cast |
| `models/staging/stg_customers.yml` | Staging schema + tests |
| `models/marts/customers.sql` | Mart: LTV aggregation + customer_type |
| `models/marts/customers.yml` | Mart schema, semantic model, metrics, saved query |
