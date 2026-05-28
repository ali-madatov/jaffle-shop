# Feature Specification: Order Intelligence

**Feature Branch**: `migrated`
**Created**: 2026-05-28
**Status**: migrated

## User Scenarios & Testing

### User Story 1 — Order Fact Table (Priority: P1)

As a business analyst, I need a single order-grain fact table that enriches
raw orders with item-level aggregations (cost, item counts, food/drink flags)
so that I can analyze order composition without writing complex joins.

**Acceptance Scenarios**:

1. **Given** an order with 2 food items and 1 drink item, **When** the
   orders mart is built, **Then** `count_food_items = 2`,
   `count_drink_items = 1`, `is_food_order = true`, `is_drink_order = true`.
2. **Given** an order with only food items, **When** the orders mart is
   built, **Then** `is_drink_order = false`.
3. **Given** order subtotals, **When** compared, **Then**
   `order_items_subtotal = subtotal` (expression test).
4. **Given** order totals, **When** compared, **Then**
   `order_total = subtotal + tax_paid` (expression test).

### User Story 2 — Order Item Detail (Priority: P1)

As a revenue analyst, I need an order-item-grain table that joins order
items with product prices, supply costs, and order timestamps so I can
compute per-item profitability.

**Acceptance Scenarios**:

1. **Given** a product with supplies costing $4.50 and $5.00, **When** the
   order_items mart is built, **Then** `supply_cost = 9.50` for that product.
2. **Given** an order item, **When** I look up its `product_price` and
   `supply_cost`, **Then** both are populated from the respective staging
   models.

### User Story 3 — Order Metrics (Priority: P1)

As a data consumer, I need semantic layer metrics for order volume, filtered
order counts (food, drink, new customer, large), and total spend.

**Acceptance Scenarios**:

1. **Given** the `orders` metric, **When** queried by day, **Then** it
   returns the count of orders per day.
2. **Given** the `food_orders` metric, **When** queried, **Then** it
   filters to orders where `is_food_order = true`.
3. **Given** the `new_customer_orders` metric, **When** queried, **Then**
   it filters to `customer_order_number = 1`.

### User Story 4 — Unit Test: Boolean Computation (Priority: P2)

As a developer, I need a unit test proving that food/drink item counts
correctly convert to boolean flags.

**Acceptance Scenarios**:

1. **Given** mock order_items with mixed food/drink, **When** the model
   computes booleans, **Then** `is_food_order` and `is_drink_order` match
   expected values per the unit test definition.

## Requirements

1. `stg_orders` renames columns and computes `subtotal = order_total - tax_paid`.
2. `stg_order_items` renames `id → order_item_id`, maps `sku → product_id`.
3. `order_items` mart joins stg_order_items + stg_orders + stg_products +
   stg_supplies; aggregates supply_cost per product.
4. `orders` mart joins stg_orders with order_items aggregations; computes
   food/drink booleans from item counts.
5. Primary keys tested with `not_null` + `unique`.
6. FK `customer_id` tested with `relationships` to `stg_customers`.
7. FK `order_id` on order_items tested with `relationships` to `orders`.
8. Two expression tests on orders: subtotal consistency, total = subtotal + tax.
9. Two unit tests: `test_order_items_compute_to_bools_correctly`,
   `test_supply_costs_sum_correctly`.
10. Semantic model `orders`: 5 dimensions, 4 measures, 6 metrics.
11. Semantic model `order_item`: 3 dimensions, 4 measures.
12. Saved query `order_metrics` exported as table with daily grain.

## Dependencies

- **Upstream**: `source('ecom', 'raw_orders')`, `source('ecom', 'raw_items')`,
  `ref('stg_products')`, `ref('stg_supplies')`
- **Downstream**: `ref('customers')` mart consumes `ref('orders')`

## Files

| File | Purpose |
|------|---------|
| `models/staging/stg_orders.sql` | Staging: rename + derive subtotal |
| `models/staging/stg_orders.yml` | Schema + expression test |
| `models/staging/stg_order_items.sql` | Staging: rename + remap SKU |
| `models/staging/stg_order_items.yml` | Schema + tests |
| `models/marts/order_items.sql` | Mart: enrich items with prices + costs |
| `models/marts/order_items.yml` | Schema, unit test, semantic model |
| `models/marts/orders.sql` | Mart: order fact with food/drink flags |
| `models/marts/orders.yml` | Schema, unit test, semantic model, metrics, saved query |
