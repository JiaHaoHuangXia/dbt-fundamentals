# dbt Fundamentals: Jaffle Shop Pipeline

[![dbt](https://img.shields.io/badge/dbt-v1.x-FF694B?style=for-the-badge&logo=dbt&logoColor=white)](https://www.getdbt.com/)
[![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Cloud-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![SQL](https://img.shields.io/badge/SQL-Analytics%20Engineering-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://en.wikipedia.org/wiki/SQL)

This repository contains the end-to-end analytics engineering data pipeline built as part of the official **dbt Fundamentals** training and certification program. It transforms raw e-commerce transaction data from a fictitious sandwich store (*Jaffle Shop*) and payment records (*Stripe*) into tested, documented, and production-grade data marts in Snowflake.

---

## 🎯 Architecture & Data Flow

The project adheres to modular analytics engineering standards by organizing transformations into distinct staging and marts layers:

```
[Raw Sources: Snowflake]
   ├── raw.jaffle_shop.customers
   ├── raw.jaffle_shop.orders
   └── raw.stripe.payment
              │
              ▼  (dbt staging layer: 1:1 view projections, renaming, typing)
   ├── stg_jaffle_shop__customers
   ├── stg_jaffle_shop__orders
   └── stg_stripe__payment
              │
              ▼  (dbt marts layer: business logic, aggregations, joins)
   ├── fct_orders (Fact table: order values, status flags, and timestamps)
   └── dim_customers (Dimension table: customer lifetime value, order history)
```

---

## 📁 Repository Structure

```text
dbt-fundamentals/
├── analyses/                  # Ad-hoc analytical queries
├── macros/                    # Custom Jinja macros for reusable query logic
├── models/
│   ├── marts/                 # Business-facing dimensional models
│   │   ├── core/
│   │   │   ├── dim_customers.sql
│   │   │   └── fct_orders.sql
│   │   └── marts.yml          # Schema definitions, docs, and test assertions
│   └── staging/               # Cleaned and standardized source data
│       ├── jaffle_shop/
│       │   ├── _jaffle_shop__sources.yml
│       │   ├── _jaffle_shop__models.yml
│       │   ├── stg_jaffle_shop__customers.sql
│       │   └── stg_jaffle_shop__orders.sql
│       └── stripe/
│           ├── _stripe__sources.yml
│           ├── _stripe__models.yml
│           └── stg_stripe__payment.sql
├── seeds/                     # Static lookup CSVs loaded into the warehouse
├── snapshots/                 # SCD Type-2 snapshots for point-in-time tracking
├── tests/                     # Singular custom assertions
│   └── assert_stg_stripe__payment_total_positive.sql
├── dbt_project.yml            # Core project configurations and materializations
└── README.md
```

---

## ⚙️ Materialization Strategy

As defined in `dbt_project.yml`:
* **Staging models (`models/staging/`):** Materialized as **Views** to keep pipelines lightweight and avoid duplicating raw storage.
* **Marts models (`models/marts/`):** Materialized as **Tables** to optimize query performance for downstream BI tools and end consumers.

---

## 🧪 Data Testing & Quality Control

Data integrity is validated using both generic schema tests and singular SQL tests:

| Test Type | Scope | Target Models / Columns | Verification Logic |
| :--- | :--- | :--- | :--- |
| **Generic** | `unique` | `customer_id`, `order_id` | Verifies primary keys have zero duplicates |
| **Generic** | `not_null` | `customer_id`, `order_id` | Ensures primary entity keys are never empty |
| **Generic** | `accepted_values` | `order_status` | Restricts values to `placed`, `shipped`, `completed`, `return_pending`, `returned` |
| **Generic** | `relationships` | `customer_id` | Enforces referential integrity between orders and customers |
| **Singular** | Custom SQL | `stg_stripe__payment` | Asserts total payment amounts per order are always greater than 0 |

Run all tests via:
```bash
dbt test
```

---

## 🚀 Getting Started

### Prerequisites
* Python 3.9+ (if using dbt Core) or an active [dbt Cloud account](https://cloud.getdbt.com/)
* A Snowflake account with permission to query the raw sample datasets

### Running Locally with dbt Core

1. **Clone the repository:**
   ```bash
   git clone https://github.com/JiaHaoHuangXia/dbt-fundamentals.git
   cd dbt-fundamentals
   ```

2. **Configure connection profile:**
   Ensure your `~/.dbt/profiles.yml` contains a target matching this project:
   ```yaml
   default:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: <your_account_identifier>
         user: <your_username>
         password: <your_password>
         role: ACCOUNTADMIN
         database: analytics
         warehouse: transforming
         schema: dbt_<your_name>
         threads: 4
   ```

3. **Verify connection and build models:**
   ```bash
   dbt debug
   dbt run
   dbt test
   ```

4. **Generate and view documentation:**
   ```bash
   dbt docs generate
   dbt docs serve
   ```

---

## 📜 Key Learnings & Skills Demonstrated

- **Modular SQL:** Writing DRY (*Don't Repeat Yourself*) code using CTE conventions and `ref()` functions.
- **Source Management:** Encapsulating upstream raw tables using `source()` references and freshness tracking.
- **Data Quality:** Proactive anomaly detection through automated assertions.
- **Documentation as Code:** Self-documenting lineage, field descriptions, and doc blocks (`{% docs %}`).
- **Version Control:** Git feature-branch workflows, pull request reviews, and remote repository integration.
