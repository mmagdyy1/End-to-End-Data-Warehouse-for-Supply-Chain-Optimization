# 🏗️ End-to-End-Data-Warehouse-for-Supply-Chain-Optimization

> End-to-end DWH integrating CRM & ERP data into a Star Schema for BI and Analytics.

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

## 🛠️ Tech Stack

<div align="center">

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
![Draw.io](https://img.shields.io/badge/Draw.io-F08705?style=for-the-badge&logo=diagramsdotnet&logoColor=white)

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

The Gold layer exposes a **Sales Data Mart** with one fact table and two dimensions:

**`gold.fact_sales`** — order_number · product_key (FK) · customer_key (FK) · order_date · shipping_date · due_date · quantity · price · sales_amount *(= quantity × price)*

**`gold.dim_customers`** — customer_key (PK) · customer_id · first_name · last_name · country · gender · birthdate · marital_status

**`gold.dim_products`** — product_key (PK) · product_id · product_name · category · subcategory · cost · product_line · maintenance · start_date

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

*Built with SQL Server · Medallion Architecture · Star Schema*
