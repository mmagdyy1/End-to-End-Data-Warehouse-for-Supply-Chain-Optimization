# 🏗️ End-to-End-Data-Warehouse-for-Supply-Chain-Optimization

> A production-grade Data Warehouse built on **SQL Server** using the **Bronze → Silver → Gold** Medallion Architecture, integrating CRM and ERP data sources into a clean **Star Schema** for BI, reporting, and machine learning.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [High-Level Architecture](#-high-level-architecture)
- [Data Flow (Lineage)](#-data-flow-data-lineage)
- [Data Integration](#-data-integration)
- [Data Model (Star Schema)](#-data-model-star-schema)
- [Tech Stack](#-tech-stack)
- [Layer Breakdown](#-layer-breakdown)
- [Source Systems](#-source-systems)
- [Gold Layer — Sales Data Mart](#-gold-layer--sales-data-mart)
- [Getting Started](#-getting-started)

---

## 📌 Project Overview

This project implements a **multi-layer Data Warehouse** that consolidates data from two source systems — **CRM** (Customer Relationship Management) and **ERP** (Enterprise Resource Planning) — into a structured, analytics-ready data mart using the **Medallion Architecture** pattern.

The pipeline is orchestrated via **SQL Server Stored Procedures** and follows a full-load, truncate-and-insert strategy for data reliability.

### Key Goals

- Centralize fragmented data from CRM and ERP into a single source of truth
- Apply structured transformations across three progressive layers (Bronze → Silver → Gold)
- Deliver a **Star Schema** data mart ready for BI dashboards, ad-hoc SQL queries, and machine learning models

---

## 🏛️ High-Level Architecture

The architecture follows the **Medallion (Lakehouse) pattern** implemented entirely within SQL Server:

```
Sources  ──►  Bronze Layer  ──►  Silver Layer  ──►  Gold Layer  ──►  Consume
(CSV Files)   (Raw Data)        (Clean Data)        (Business-Ready)  (BI / ML)
```

![High Level Architecture](./docs/images/data_architecture.png)

| Layer | Object Type | Load Strategy | Transformations | Data Model |
|-------|-------------|---------------|-----------------|------------|
| **Bronze** | Tables | Batch / Full Load / Truncate & Insert | None (as-is) | None |
| **Silver** | Tables | Batch / Full Load / Truncate & Insert | Cleansing, Standardization, Normalization, Derived Columns, Enrichment | None (as-is) |
| **Gold** | Views | No Load | Data Integrations, Aggregations, Business Logics | Star Schema / Flat Table / Aggregated Table |

---

## 🔄 Data Flow (Data Lineage)

The diagram below shows how each source table flows through all three layers and which Gold layer objects they contribute to:

![Data Flow](./docs/images/data_flow.png)

### Lineage Summary

**CRM Source Tables:**

| Bronze | Silver | Gold Output |
|--------|--------|-------------|
| `crm_sales_details` | `crm_sales_details` | `fact_sales` |
| `crm_cust_info` | `crm_cust_info` | `dim_customers` |
| `crm_prd_info` | `crm_prd_info` | `dim_products` |

**ERP Source Tables:**

| Bronze | Silver | Gold Output |
|--------|--------|-------------|
| `erp_cust_az12` | `erp_cust_az12` | `dim_customers` |
| `erp_loc_a101` | `erp_loc_a101` | `dim_customers` |
| `erp_px_cat_g1v2` | `erp_px_cat_g1v2` | `dim_products` |

> **Note:** Both `dim_customers` and `dim_products` are enriched by merging CRM and ERP data during Gold layer view creation.

---

## 🔗 Data Integration

This diagram shows the relationships between source tables across CRM and ERP systems, and how they link to each other:

![Data Integration](./docs/images/data_integration.png)

### CRM Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `crm_sales_details` | Transactional records for Sales & Orders | `prd_key`, `cst_id` |
| `crm_cust_info` | Customer Information | `cst_id`, `cst_key` |
| `crm_prd_info` | Current & Historical Product Information | `prd_key` |

### ERP Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `erp_cust_az12` | Extra Customer Information (Birthdate) | `cid` |
| `erp_loc_a101` | Location of Customers (Country) | `cid` |
| `erp_px_cat_g1v2` | Product Categories | `id` |

### Integration Keys

- `crm_sales_details.prd_key` → `crm_prd_info.prd_key` *(Sales ↔ Product)*
- `crm_sales_details.cst_id` → `crm_cust_info.cst_id` *(Sales ↔ Customer)*
- `crm_cust_info.cst_key` → `erp_cust_az12.cid` *(CRM Customer ↔ ERP Customer Info)*
- `crm_cust_info.cst_key` → `erp_loc_a101.cid` *(CRM Customer ↔ ERP Location)*
- `crm_prd_info.PRODUCT` → `erp_px_cat_g1v2.PRODUCT` *(CRM Product ↔ ERP Categories)*

---

## 📐 Data Model (Star Schema)

The Gold Layer is organized as a **Sales Data Mart** following the Star Schema pattern:

![Data Model](./docs/images/data_model.png)

### `gold.fact_sales` — Fact Table

| Column | Description |
|--------|-------------|
| `order_number` | Unique order identifier |
| `product_key` (FK1) | FK → `gold.dim_products` |
| `customer_key` (FK2) | FK → `gold.dim_customers` |
| `order_date` | Date the order was placed |
| `shipping_date` | Date the order was shipped |
| `due_date` | Expected delivery date |
| `sales_amount` | Derived: `quantity * price` |
| `quantity` | Number of units ordered |
| `price` | Unit price |

> **Sales Calculation:** `sales_amount = quantity × price`

---

### `gold.dim_customers` — Customer Dimension

| Column | Description |
|--------|-------------|
| `customer_key` (PK) | Surrogate key |
| `customer_id` | Source system customer ID |
| `customer_number` | Business customer number |
| `first_name` | First name |
| `last_name` | Last name |
| `country` | Customer country (from ERP) |
| `marital_status` | `Married` or `Single` |
| `gender` | Gender |
| `birthdate` | Date of birth (from ERP) |

---

### `gold.dim_products` — Product Dimension

| Column | Description |
|--------|-------------|
| `product_key` (PK) | Surrogate key |
| `product_id` | Source system product ID |
| `product_number` | Business product number |
| `product_name` | Full product name |
| `category_id` | Product category ID |
| `category` | Category name |
| `subcategory` | Subcategory name |
| `maintenance` | `Yes` or `No` |
| `cost` | Product cost |
| `product_line` | Product line |
| `start_date` | Product availability start date |

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| **Database** | Microsoft SQL Server |
| **ETL Mechanism** | Stored Procedures |
| **Load Strategy** | Full Load — Truncate & Insert |
| **Processing** | Batch Processing |
| **Data Model** | Star Schema (Fact + Dimensions) |
| **Gold Objects** | SQL Views (no physical storage) |
| **Consumers** | Power BI, Ad-Hoc SQL, Machine Learning |

---

## 📦 Layer Breakdown

### 🥉 Bronze Layer — Raw Ingestion

- **Purpose:** Land raw data from source systems as-is, with no transformation
- **Object Type:** Tables
- **Load:** Batch, Full Load, Truncate & Insert
- **Transformations:** None
- **Data Model:** None (exact source replica)
- **Orchestration:** SQL Server Stored Procedures

### 🥈 Silver Layer — Cleansed & Standardized

- **Purpose:** Clean, standardize, and enrich raw data for downstream use
- **Object Type:** Tables
- **Load:** Batch, Full Load, Truncate & Insert
- **Transformations:**
  - Data Cleansing (nulls, duplicates, invalid values)
  - Data Standardization (formats, codes, enumerations)
  - Data Normalization (consistent units and scales)
  - Derived Columns (calculated fields)
  - Data Enrichment (additional context and lookups)
- **Data Model:** None (as-is after transformation)

### 🥇 Gold Layer — Business-Ready

- **Purpose:** Deliver analytics-ready, business-logic-applied views for consumption
- **Object Type:** Views (no physical load)
- **Transformations:**
  - Data Integrations (joining CRM + ERP data)
  - Aggregations (pre-computed measures)
  - Business Logics (sales calculations, categorizations)
- **Data Model:** Star Schema, Flat Tables, Aggregated Tables

---

## 📂 Source Systems

### CRM (Customer Relationship Management)

- **Interface:** CSV Files in Folders
- **Object Type:** CSV Files
- **Tables Loaded:** `crm_sales_details`, `crm_cust_info`, `crm_prd_info`

### ERP (Enterprise Resource Planning)

- **Interface:** CSV Files in Folders
- **Object Type:** CSV Files
- **Tables Loaded:** `erp_cust_az12`, `erp_loc_a101`, `erp_px_cat_g1v2`

---

## 🌟 Gold Layer — Sales Data Mart

The final output is a **Sales Data Mart** structured as a Star Schema with:

- **1 Fact Table:** `gold.fact_sales` — transactional sales records
- **2 Dimension Tables:**
  - `gold.dim_customers` — enriched customer profiles (CRM + ERP merged)
  - `gold.dim_products` — enriched product catalog (CRM + ERP merged)

### Consumption Use Cases

| Consumer | Description |
|----------|-------------|
| **BI & Reporting** | Power BI dashboards and reports connected to Gold views |
| **Ad-Hoc SQL Queries** | Analysts running direct SQL queries on Gold views |
| **Machine Learning** | Feature engineering and model training datasets |

---


## 📁 Project Structure

```
sales-dwh/
├── docs/
│   └── images/
│       ├── data_architecture.png
│       ├── data_flow.png
│       ├── data_integration.png
│       └── data_model.png
├── bronze/
│   └── stored_procedures/
├── silver/
│   └── stored_procedures/
├── gold/
│   └── views/
├── scripts/
│   └── init_database.sql
└── README.md
```

---

## 📊 Data Quality Notes

- All loads use **Truncate & Insert** to ensure idempotency
- The Bronze layer preserves source data as-is — no data is lost or modified at ingestion
- The Silver layer handles all data quality issues before data reaches the Gold layer
- Gold layer views are computed on-the-fly — no stale data risk
