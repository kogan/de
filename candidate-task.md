# Data engineering take-home — dbt (mid-level)

**Time:** 60 minutes (40 minutes coding)
**Stack:** dbt (Cloud) against BigQuery (or your local database)

---

## Context

You join a small e-commerce team. An upstream system exports **append-only snapshots** of order headers and line items. The exports are **not** idempotent: the same logical order appears on many rows, and fields can be **partially null** on newer rows. Finance needs a **single current row per order** and a **reliable line-total** to reconcile against the header (order).

Your job is to implement this in **dbt** with optimisation and quality controls.

---

## Source data (provided)

### `raw_orders.csv`

| Column               | Description                                                                         |
| -------------------- | ----------------------------------------------------------------------------------- |
| `order_id`           | Business key of the order                                                           |
| `customer_id`        | Customer identifier                                                                 |
| `extracted_at`       | When this snapshot row was produced (ISO-like timestamp string)                     |
| `order_status`       | e.g. `placed`, `paid`, `shipped`, `cancelled`, `refunded`                           |
| `order_total_amount` | Money as decimal; **may be null** on some later rows even when still the same order |

### `raw_line_items.csv`

| Column         | Description                                                     |
| -------------- | --------------------------------------------------------------- |
| `order_id`     | Foreign key to order                                            |
| `sku`          | Product sku                                                     |
| `qty`          | Integer quantity                                                |
| `unit_price`   | Unit price (decimal)                                            |
| `line_no`      | Integer line sequence from source (not globally unique)         |
| `extracted_at` | When this snapshot row was produced (ISO-like timestamp string) |

---

## Requirements

1. **From seeds**
   - `dbt seed` - to populate raw_orders and raw_line_items tables

2. **(Optional) The intermetiates**
   - Implement **one row per `order_id`** reflecting:
     - the **latest** **status** \*.
     - **amount** = rule above (latest non-null amount carried backward per order).
   - Implement **grouped line items** at grain **`order_id`, `sku`** with **`qty` summed**.

3. **Marts**
   - Model **`fct_orders`** at grain **one row per `order_id`** with:
     - `order_id`, `customer_id`, `order_status`
     - `header_amount` (from your deduped / filled header logic)
     - `lines_subtotal` = sum over lines of `qty * unit_price` for that order
     - total order value should match total order line value

4. **Tests**

- any tests you think are mandatory or reasonable

---

## Out of scope

- Orchestration, Airflow, infra.
- Real-time APIs; work from seeds only.

Good luck.
