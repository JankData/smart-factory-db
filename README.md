# 🏭 Smart Factory Database

A relational database project modelling the operations of a modern, AI-assisted robotics manufacturing plant. Designed as part of a **Databases** university course.

**Authors:** Magdalena Chamera & Karol Jank

---

## Overview

This project implements a fully normalised PostgreSQL database that simulates the core business processes of a smart factory producing industrial robots and automation equipment. The schema covers the complete production lifecycle — from component procurement and inventory management, through machine scheduling and order execution, to AI-powered quality inspection and maintenance tracking.

---

## Schema

The database consists of **10 tables**, **2 views**, **2 functions**, and **1 trigger**.

### Tables

| Table | Description |
|---|---|
| `products` | Finished goods manufactured at the plant (drones, cobots, AMRs, etc.) |
| `components` | Raw components and sub-assemblies used in production |
| `bill_of_materials` | Many-to-many mapping of which components are required per product |
| `suppliers` | External vendors supplying components |
| `supply_catalog` | Per-supplier pricing and lead times for each component |
| `machines` | Factory machines (robotic arms, CNC mills, 3D printers, laser cutters) |
| `employees` | Factory staff with job roles and certification levels (1–5) |
| `production_orders` | Scheduled and completed manufacturing runs, assigned to a machine and employee |
| `maintenance_logs` | Records of machine servicing, linked to the responsible technician |
| `ai_quality_inspections` | Automated inspection results (PASS / FAIL / ANOMALY) with confidence scores |

### Views

| View | Description |
|---|---|
| `low_stock_alerts` | Lists components at or below their reorder point, together with the cheapest available supplier and lead time |
| `production_efficiency` | Per-machine summary of total orders run, passed inspections, and average AI confidence score |

### Functions & Triggers

| Object | Type | Description |
|---|---|---|
| `calculate_product_cost(p_id)` | Function | Returns the minimum achievable BOM cost for a given product by selecting the lowest unit price across all suppliers for each required component |
| `check_machine_overlap()` | Trigger function | Raises an exception if a new or updated production order would double-book a machine during an already-occupied time slot |
| `trg_check_machine_availability` | Trigger | Fires `BEFORE INSERT OR UPDATE` on `production_orders` to enforce machine scheduling integrity |

---

## Entity-Relationship Overview

```
suppliers ──< supply_catalog >── components ──< bill_of_materials >── products
                                                                          │
                                                               production_orders
                                                              /         |        \
                                                        machines   employees   ai_quality_inspections
                                                            │
                                                     maintenance_logs
                                                            │
                                                        employees
```

---

## Sample Data

The dump includes realistic seed data across all tables:

- **6 products** — Industrial Drone X-1, Cobot Arm Pro, AMR Logistic Bot, Inspection Quadruped, Automated Welding Cell, Micro-Assembly Robot
- **15 components** — across Sensor, Actuator, Computing, Structural, and Electronic categories
- **8 suppliers** — from Germany, USA, China, Japan, UK, Poland, and Canada
- **8 machines** — 3 robotic arms, 2 CNC mills, 2 3D printers, 1 laser cutter
- **10 employees** — spanning roles from Robotics Engineer to Assembly Line Worker
- **15 production orders** — with Completed, In Progress, and Scheduled statuses
- **11 AI quality inspection records**

---

## Setup & Usage

**Requirements:** PostgreSQL 14 or higher (the dump was generated with version 18.3).

**Restore the database:**

```bash
# Create a target database
createdb smart_factory

# Load the schema and seed data
psql -d smart_factory -f factory.sql
```

**Verify the installation:**

```sql
-- Check all tables are present
\dt

-- Preview products and their minimum production cost
SELECT name, model_version, base_price,
       calculate_product_cost(product_id) AS min_bom_cost
FROM products;

-- Check which components need reordering
SELECT * FROM low_stock_alerts;

-- Review machine efficiency metrics
SELECT * FROM production_efficiency;
```

---

## Key Design Decisions

- **Scheduling integrity** is enforced at the database level via a `BEFORE INSERT OR UPDATE` trigger, preventing any machine from being assigned two overlapping production orders.
- **Cost optimisation** is built into the `calculate_product_cost` function, which always selects the cheapest available supplier for each BOM line item, providing a realistic lower-bound production cost.
- **AI quality control** is modelled as a first-class entity, with confidence scores and a three-state result (PASS / FAIL / ANOMALY), reflecting real Industry 4.0 inspection pipelines.
- **Inventory alerts** are surfaced through a view rather than application logic, keeping the business rule close to the data.
- **Cascading deletes** are applied on `bill_of_materials` and `supply_catalog` so that removing a component automatically cleans up dependent records.

---

## Technologies

- **DBMS:** PostgreSQL
- **Language:** PL/pgSQL (stored functions and triggers)
- **Dump tool:** `pg_dump`

---

## License

This project was created for educational purposes as part of a university course assignment.
