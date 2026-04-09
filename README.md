# 🏗️ End-to-End-Data-Warehouse-for-Supply-Chain-Optimization

### 🧠 End-to-End Data Engineering Project | SQL | Data Warehouse | ETL

![SQL Server](https://img.shields.io/badge/Language-SQL-yellow?style=flat-square)
![ETL](https://img.shields.io/badge/Process-ETL-green?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Medallion-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

---

## 🏭 Problem Statement

Organizations typically manage their operations through two separate systems:

- **CRM systems** → handle sales, customers, and orders.
- **ERP systems** → handle products, pricing, and locations.

However, these systems operate in **silos**, making it difficult to:

- Get a **unified view** of sales performance
- Enrich customer data with **location and demographic info**
- Link products to their **categories and cost structure**
- Build reliable **BI dashboards or ML models**

---

## 🎯 Project Goal

Design and implement an **end-to-end data pipeline** that integrates CRM and ERP data into a **centralized data warehouse**, enabling:

✅ Clean, unified, analytics-ready data  
✅ Full data lineage traceability (Bronze → Silver → Gold)  
✅ A Star Schema data mart optimized for BI and reporting  
✅ Scalable architecture built entirely on SQL Server  

---

## 📌 Project Overview

This project builds a modern **Sales Data Warehouse** on **Microsoft SQL Server**, consolidating data from two source systems — a **CRM** and an **ERP** — into a clean, analytics-ready data mart using the **Medallion Architecture** (Bronze → Silver → Gold).

### 📥 Data Sources

Both systems deliver data as **CSV files** dropped into folders, loaded via SQL Server Stored Procedures.

| Source | Full Name | Tables | Contains |
|--------|-----------|--------|----------|
| **CRM** | Customer Relationship Management | `crm_sales_details`, `crm_cust_info`, `crm_prd_info` | Sales transactions, customer profiles, product info |
| **ERP** | Enterprise Resource Planning | `erp_cust_az12`, `erp_loc_a101`, `erp_px_cat_g1v2` | Customer birthdates, customer locations, product categories |

### ⚙️ How It Works

```
CSV Files → [Bronze] Raw Ingest → [Silver] Clean & Standardize → [Gold] Business Logic → BI / ML
```

| Step | Layer | What Happens |
|------|-------|--------------|
| 1 | 🥉 **Bronze** | CSV files loaded as-is into SQL Server tables — pure raw landing zone, no changes. |
| 2 | 🥈 **Silver** | Data cleansed (nulls, duplicates), standardized (formats, codes), normalized, enriched, derived columns computed. |
| 3 | 🥇 **Gold** | CRM & ERP integrated, business logic applied via SQL Views — final Star Schema exposed. |

All loads use a **Truncate & Insert (Full Load)** strategy via **Stored Procedures**.

### 📊 Final Output — Sales Data Mart

| Object | Type | Key Columns |
|--------|------|-------------|
| `gold.fact_sales` | Fact Table | `order_number`, `product_key` (FK), `customer_key` (FK), `order_date`, `quantity`, `price`, `sales_amount` |
| `gold.dim_customers` | Dimension | `customer_key` (PK), `first_name`, `last_name`, `country`, `gender`, `birthdate`, `marital_status` |
| `gold.dim_products` | Dimension | `product_key` (PK), `product_name`, `category`, `subcategory`, `cost`, `product_line`, `maintenance` |

> **Sales Calculation:** `sales_amount = quantity × price`

### 🎁 Who Is This For?

| Role | How They Use It |
|------|----------------|
| **BI Analyst** | Connect Power BI to Gold views for dashboards & reports |
| **Data Analyst** | Run ad-hoc SQL queries on a clean, well-modeled schema |
| **Data Scientist** | Use Gold layer as a feature store for ML pipelines |
| **Data Engineer** | Reference as a Medallion Architecture template on SQL Server |

---

## 🛠️ Tech Stack

<div align="center">

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
![Draw.io](https://img.shields.io/badge/Draw.io-F08705?style=for-the-badge&logo=diagramsdotnet&logoColor=white)
![Notion](https://img.shields.io/badge/Notion-000000?style=for-the-badge&logo=notion&logoColor=white)

</div>

---

## 🏛️ High-Level Architecture

The warehouse follows the **Medallion Architecture** pattern with three progressive layers:

![High Level Architecture](./docs/images/data_architecture.png)

| Layer | Storage | Load Strategy | Purpose |
|-------|---------|---------------|---------|
| 🥉 **Bronze** | Tables | Truncate & Insert | Raw data, no changes |
| 🥈 **Silver** | Tables | Truncate & Insert | Cleaned & standardized |
| 🥇 **Gold** | Views | No Load | Business-ready, Star Schema |

---

## 🔄 Data Flow

![Data Flow](./docs/images/data_flow.png)

| Source | Bronze → Silver | Gold Output |
|--------|-----------------|-------------|
| `crm_sales_details` | ✅ | `fact_sales` |
| `crm_cust_info` | ✅ | `dim_customers` |
| `crm_prd_info` | ✅ | `dim_products` |
| `erp_cust_az12` | ✅ | `dim_customers` |
| `erp_loc_a101` | ✅ | `dim_customers` |
| `erp_px_cat_g1v2` | ✅ | `dim_products` |

---

## 🔗 Data Integration

CRM and ERP tables are linked via business keys to build unified dimension tables.

![Data Integration](./docs/images/data_integration.png)

- `crm_sales_details.prd_key` → `crm_prd_info.prd_key`
- `crm_sales_details.cst_id` → `crm_cust_info.cst_id`
- `crm_cust_info.cst_key` → `erp_cust_az12.cid` / `erp_loc_a101.cid`
- `crm_prd_info.PRODUCT` → `erp_px_cat_g1v2.PRODUCT`

---

## 📐 Data Model — Star Schema

![Data Model](./docs/images/data_model.png)

**`gold.fact_sales`** — order_number · product_key (FK) · customer_key (FK) · order_date · shipping_date · due_date · quantity · price · sales_amount *(= quantity × price)*

**`gold.dim_customers`** — customer_key (PK) · customer_id · first_name · last_name · country · gender · birthdate · marital_status

**`gold.dim_products`** — product_key (PK) · product_id · product_name · category · subcategory · cost · product_line · maintenance · start_date

---

## 🧠 Key Features

- ✅ **Medallion Architecture** (Bronze → Silver → Gold)
- ✅ **Data Cleansing & Validation** — nulls, duplicates, invalid values
- ✅ **Schema Standardization** — type casting, date formats, trimming
- ✅ **Cross-System Integration** — CRM & ERP unified via business keys
- ✅ **Star Schema Design** — optimized for BI and analytics queries
- ✅ **Full Data Lineage** — traceable from raw CSV to Gold view
- ✅ **Idempotent Loads** — safe to re-run via Truncate & Insert

---

## 📊 Example Use Cases

- 🧾 **Sales Performance** → Total revenue by customer, product, or time period
- 👥 **Customer Insights** → Demographics, location, and marital status analysis
- 📦 **Product Analysis** → Category/subcategory performance and cost breakdown
- 🌍 **Regional Reporting** → Sales distribution by country

---

## 🗂️ Project Structure

```
sales-dwh/
├── datasets/                   # Raw source CSV files
│   ├── source_crm/
│   └── source_erp/
├── docs/
│   └── images/                 # Architecture diagrams
├── scripts/
│   ├── bronze/                 # Raw ingestion stored procedures
│   ├── silver/                 # Cleansing & transformation stored procedures
│   ├── gold/                   # Business-ready views
│   └── init_database.sql       # DB + schema setup
└── README.md
```

---

## 🚀 Getting Started

**1. Clone the repo**
```bash
git clone https://github.com/your-username/sales-dwh.git
```

**2. Create schemas**
```sql
CREATE SCHEMA bronze;
CREATE SCHEMA silver;
CREATE SCHEMA gold;
```

**3. Load Bronze → Silver → Gold**
```sql
EXEC bronze.load_all;
EXEC silver.load_all;
-- Gold views auto-apply on query
```

**4. Query the data mart**
```sql
SELECT * FROM gold.fact_sales;
SELECT * FROM gold.dim_customers;
SELECT * FROM gold.dim_products;
```

---
