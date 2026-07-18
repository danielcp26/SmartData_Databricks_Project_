# E-COMMERCE SALES, CUSTOMERS, AND DELIVERY ETL

## Medallion Architecture in Azure Databricks

Data engineering pipeline designed to integrate, clean, enrich, and aggregate e-commerce orders, customers, products, payments, and delivery data using a **Bronze → Silver → Golden** Medallion Architecture in Azure Databricks.

---

## 🎯 Description

This project implements an ETL pipeline using CSV files from the Olist e-commerce dataset. The data is stored in Azure Data Lake Storage Gen2, processed with PySpark, and published as Delta tables managed through Unity Catalog.

The final analytical layer supports analysis of:

- Order and customer volume
- Units sold
- Revenue, freight, and payment values
- Product categories
- Geographic distribution by state and region
- Payment methods
- Delivery duration and performance
- Late orders and late-delivery rates

---

## ✨ Main Features

- 🔄 **Automated ETL:** Separate notebooks for environment preparation, ingestion, transformation, and loading
- 🏗️ **Medallion Architecture:** Clear separation between Bronze, Silver, and Golden layers
- ☁️ **Azure Data Lake Storage Gen2:** Physical storage through `abfss` paths
- ⚡ **Delta Lake:** Delta tables with ACID transactions
- 🗂️ **Unity Catalog:** Centralized management of catalogs, schemas, tables, and external locations
- 🧹 **Data Quality:** Explicit schemas, normalization, filtering, deduplication, and validation
- 🌎 **Geographic Enrichment:** Customer classification by Brazilian region
- 📊 **Analytical Layer:** Business aggregations for sales, payments, customers, and deliveries
- 🔐 **Access Control:** Role-based permissions for `DEs`, `CTs`, and `BI`
- 🚀 **CI/CD:** Notebook deployment between Databricks workspaces through GitHub Actions
- ♻️ **Reversion:** Notebooks for revoking permissions and deleting project objects

---

## 🏛️ Architecture

### Data Flow

```text
📄 CSV Files in ADLS Gen2
            ↓
🥉 Bronze Layer
   Raw ingestion and explicit typing
            ↓
🥈 Silver Layer
   Cleaning, integration, and enrichment
            ↓
🥇 Golden Layer
   Business metrics and aggregations
            ↓
📊 Analytical Consumption and Dashboards
```

### Pipeline Layers

#### 🥉 Bronze Layer

The Bronze layer stores source data using explicit schemas and includes an `ingestion_date` audit column.

Tables:

```text
catalog_au.bronze.orders_raw
catalog_au.bronze.customers_raw
catalog_au.bronze.order_items_raw
catalog_au.bronze.products_raw
catalog_au.bronze.payments_raw
catalog_au.bronze.category_translation_raw
```

Input files:

```text
olist_orders_dataset.csv
olist_customers_dataset.csv
olist_order_items_dataset.csv
olist_products_dataset.csv
olist_order_payments_dataset.csv
product_category_name_translation.csv
```

#### 🥈 Silver Layer

The Silver layer combines the six Bronze tables into one enriched table:

```text
catalog_au.silver.olist_orders_enriched
```

Main transformations:

- Standardization of states, cities, product categories, and payment methods
- Duplicate removal
- Filtering of invalid prices, freight values, and payment values
- Translation of product categories
- Product-volume calculation
- Payment consolidation by order
- Geographic classification by region
- Delivery-duration calculation
- Late-delivery identification
- Delivery classification as `On Time`, `Late`, or `Not Delivered`

#### 🥇 Golden Layer

The Golden layer publishes the following analytical table:

```text
catalog_au.golden.olist_sales_summary
```

Aggregation dimensions:

```text
year
month
year_month
customer_state
customer_region
product_category_name_english
payment_type
delivery_status
```

Metrics:

```text
total_orders
total_customers
total_items_sold
total_revenue
total_freight
total_payment_value
average_item_price
average_payment_value
average_delivery_days
late_orders
late_delivery_rate
```

---

## 📁 Project Structure

```text
SmartData_Databricks_Project_
├── .github/
│   └── workflows/
├── Enviroment_Prep/
│   └── 1. Enviroment_Preparation
├── Process/
│   ├── 2. Ingest_Category_Translation
│   ├── 2. Ingest_Customers
│   ├── 2. Ingest_Order_Items
│   ├── 2. Ingest_Orders
│   ├── 2. Ingest_Payments
│   ├── 2. Ingest_Products
│   ├── 3. Transform
│   └── 4. Load
├── Reversion/
│   ├── Drop_tables
│   └── revoke
├── Security/
│   └── grant
├── Certifications/
├── Datasets/
├── Proof/
└── README.md
```

---

## 🚀 Installation and Configuration

### 1️⃣ Clone the Repository

```bash
git clone <REPOSITORY_URL>
cd SmartData_Databricks_Project_
```

Replace `<REPOSITORY_URL>` with the actual repository URL.

### 2️⃣ Configure Azure Data Lake Storage

The project uses the following storage account:

```text
adlsssmartdata2698
```

Expected containers:

```text
raw
bronze
silver
golden
metastore
```

Path format:

```text
abfss://<container>@adlsssmartdata2698.dfs.core.windows.net/
```

The six source CSV files must be available in the `raw` container.

### 3️⃣ Configure Unity Catalog

Main objects:

```text
Catalog: catalog_au

Schemas:
- raw
- bronze
- silver
- golden
```

External locations:

```text
exlt-metastore
exlt-raw
exlt-bronze
exlt-silver
exlt-golden
```

### 4️⃣ Configure Databricks Tokens

In each workspace:

```text
User Settings
→ Developer
→ Access tokens
→ Generate new token
```

### 5️⃣ Configure GitHub Secrets

In GitHub:

```text
Settings
→ Secrets and variables
→ Actions
```

Secrets used by the deployment workflow:

| Secret | Description |
|---|---|
| `DATABRICKS_ORIGIN_HOST` | Origin workspace URL |
| `DATABRICKS_ORIGIN_TOKEN` | Origin workspace access token |
| `DATABRICKS_DEST_HOST` | Destination workspace URL |
| `DATABRICKS_DEST_TOKEN` | Destination workspace access token |

Example host:

```text
https://adb-xxxxxxxxxxxxxxxx.x.azuredatabricks.net
```

Do not include a trailing `/` in the host value.

---

## 💻 Usage

### 🔧 Manual Execution in Databricks

Run the notebooks in the following order:

```text
1. Enviroment_Prep/1. Enviroment_Preparation
   → Creates the catalog, schemas, external locations, and Delta tables.

2. Process/2. Ingest_Category_Translation
   → Loads product-category translations into Bronze.

3. Process/2. Ingest_Customers
   → Loads customer data into Bronze.

4. Process/2. Ingest_Order_Items
   → Loads order-item data into Bronze.

5. Process/2. Ingest_Orders
   → Loads order data into Bronze.

6. Process/2. Ingest_Payments
   → Loads payment data into Bronze.

7. Process/2. Ingest_Products
   → Loads product data into Bronze.

8. Process/3. Transform
   → Cleans, integrates, enriches, and publishes the Silver table.

9. Process/4. Load
   → Aggregates business metrics and publishes the Golden table.
```

The ingestion notebooks may be executed in parallel after the environment-preparation notebook has completed successfully.

### 🔄 Deployment with GitHub Actions

The configured workflow deploys notebooks to the destination workspace whenever a push is made to the `main` branch.

```bash
git add .
git commit -m "Update Databricks pipeline"
git push origin main
```

General deployment flow:

```text
Repository checkout
        ↓
Notebook export from the origin workspace
        ↓
Notebook content encoding
        ↓
Notebook import into the destination workspace
        ↓
Temporary file cleanup
```

Workflow name:

```text
Dynamic Databricks Notebook Deploy
```

---

## 🔐 Security and Access Control

The project uses three account groups:

| Group | Role |
|---|---|
| `DEs` | Data Engineers |
| `CTs` | Technical Consultants |
| `BI` | BI Analysts |

### Data Engineers

Data Engineers receive broad catalog access, schema creation and usage permissions, table read access, and read/write access to external locations.

### Technical Consultants

Technical Consultants receive catalog and schema usage permissions, read access to Bronze, Silver, and Golden tables, and read access to external locations.

### BI Analysts

BI Analysts receive limited access to the catalog, the Golden schema, and the following table:

```text
catalog_au.golden.olist_sales_summary
```

Related notebooks:

```text
Security/grant
Reversion/revoke
```

---

## ♻️ Reversion

### Revoke Permissions

Run:

```text
Reversion/revoke
```

This notebook removes the privileges assigned to `DEs`, `CTs`, and `BI`.

### Delete Project Objects

Run:

```text
Reversion/Drop_tables
```

This notebook contains instructions for deleting project tables, schemas, the catalog, and external locations. Commands that physically delete data from ADLS should remain commented out unless permanent deletion is explicitly required.

---

## 📈 Validation Queries

### Silver Table

```sql
SELECT *
FROM catalog_au.silver.olist_orders_enriched
LIMIT 10;
```

### Golden Table

```sql
SELECT *
FROM catalog_au.golden.olist_sales_summary
LIMIT 10;
```

### Monthly Summary

```sql
SELECT
    year_month,
    SUM(total_orders) AS orders,
    SUM(total_revenue) AS revenue,
    AVG(late_delivery_rate) AS average_late_delivery_rate
FROM catalog_au.golden.olist_sales_summary
GROUP BY year_month
ORDER BY year_month;
```

---

## 🔍 Monitoring

### In Databricks

- Review the output of each notebook cell
- Validate record counts in Bronze, Silver, and Golden
- Review Delta table history
- Check for read, write, or permission errors
- Review pipeline execution under **Workflows** when configured as a Databricks Job

### In GitHub Actions

- Open the **Actions** tab
- Select `Dynamic Databricks Notebook Deploy`
- Open a workflow execution
- Review the export and import logs for each notebook

---

## 🧰 Technologies

```text
Azure Databricks
Apache Spark
PySpark
Spark SQL
Delta Lake
Unity Catalog
Azure Data Lake Storage Gen2
GitHub Actions
Git
```

---

## 👤 Author

**Daniel Chacón**

Data Engineering | Azure Databricks | PySpark | Delta Lake | Unity Catalog | CI/CD

---

## 📄 License

Add a `LICENSE` file if the project will be distributed under a specific license.

---

**Project:** Data Engineering with Medallion Architecture  
**Technologies:** Azure Databricks + PySpark + Delta Lake + ADLS Gen2 + Unity Catalog + GitHub Actions
