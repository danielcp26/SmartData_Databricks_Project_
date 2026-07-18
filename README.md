# SmartData_Databricks_Project_

# ETL DE VENTAS, CLIENTES Y ENTREGAS DE E-COMMERCE

## Arquitectura Medallion en Azure Databricks

Pipeline de ingeniería de datos para integrar, limpiar y agregar información de pedidos, clientes, productos, pagos y entregas de e-commerce mediante una arquitectura Medallion **Bronze → Silver → Golden** en Azure Databricks.

---

## 🎯 Descripción

Este proyecto implementa un pipeline ETL sobre archivos CSV del conjunto de datos de e-commerce Olist. Los datos se almacenan en Azure Data Lake Storage Gen2, se procesan con PySpark y se publican como tablas Delta administradas mediante Unity Catalog.

El resultado final es una tabla analítica que permite estudiar:

- Volumen de pedidos y clientes.
- Unidades vendidas.
- Ingresos, fletes y pagos.
- Categorías de productos.
- Distribución geográfica por estado y región.
- Métodos de pago.
- Duración y cumplimiento de entregas.
- Pedidos tardíos y tasa de retraso.

---

## ✨ Características principales

- 🔄 **ETL automatizado:** notebooks independientes para preparación, ingesta, transformación y carga.
- 🏗️ **Arquitectura Medallion:** separación clara entre las capas Bronze, Silver y Golden.
- ☁️ **Azure Data Lake Storage Gen2:** almacenamiento físico mediante rutas `abfss`.
- ⚡ **Delta Lake:** tablas Delta con transacciones ACID.
- 🗂️ **Unity Catalog:** catálogo, esquemas, tablas y external locations centralizados.
- 🧹 **Calidad de datos:** tipado explícito, normalización, filtros, deduplicación y validaciones.
- 🌎 **Enriquecimiento geográfico:** clasificación de clientes por región de Brasil.
- 📊 **Capa analítica:** agregaciones de ventas, pagos, clientes y entregas.
- 🔐 **Control de acceso:** permisos diferenciados para `DEs`, `CTs` y `BI`.
- 🚀 **CI/CD:** despliegue de notebooks entre workspaces mediante GitHub Actions.
- ♻️ **Reversión:** notebooks para revocar permisos y eliminar objetos del proyecto.

---

## 🏛️ Arquitectura

### Flujo de datos

```text
📄 Archivos CSV en ADLS Gen2
            ↓
🥉 Bronze Layer
   Ingesta y tipado de datos
            ↓
🥈 Silver Layer
   Limpieza, unificación y enriquecimiento
            ↓
🥇 Golden Layer
   Métricas y agregaciones de negocio
            ↓
📊 Consumo analítico y dashboards
```

### Capas del pipeline

#### 🥉 Bronze

La capa Bronze conserva los datos de origen con un esquema explícito y una columna de auditoría `ingestion_date`.

Tablas:

```text
catalog_au.bronze.orders_raw
catalog_au.bronze.customers_raw
catalog_au.bronze.order_items_raw
catalog_au.bronze.products_raw
catalog_au.bronze.payments_raw
catalog_au.bronze.category_translation_raw
```

Archivos de entrada:

```text
olist_orders_dataset.csv
olist_customers_dataset.csv
olist_order_items_dataset.csv
olist_products_dataset.csv
olist_order_payments_dataset.csv
product_category_name_translation.csv
```

#### 🥈 Silver

La capa Silver combina las seis tablas Bronze en una tabla enriquecida:

```text
catalog_au.silver.olist_orders_enriched
```

Transformaciones principales:

- Normalización de estados, ciudades, categorías y métodos de pago.
- Eliminación de duplicados.
- Filtrado de precios, fletes y pagos inválidos.
- Traducción de categorías de productos.
- Cálculo del volumen del producto.
- Consolidación de pagos por pedido.
- Clasificación geográfica por región.
- Cálculo de días de entrega.
- Identificación de entregas tardías.
- Clasificación de entrega: `On Time`, `Late` o `Not Delivered`.

#### 🥇 Golden

La capa Golden publica la tabla:

```text
catalog_au.golden.olist_sales_summary
```

Dimensiones de agregación:

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

Métricas:

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

## 📁 Estructura del proyecto

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

## 🚀 Instalación y configuración

### 1️⃣ Clonar el repositorio

```bash
git clone <URL_DEL_REPOSITORIO>
cd SmartData_Databricks_Project_
```

Sustituya `<URL_DEL_REPOSITORIO>` por la URL real del repositorio.

### 2️⃣ Configurar Azure Data Lake Storage

El proyecto utiliza la cuenta:

```text
adlsssmartdata2698
```

Contenedores esperados:

```text
raw
bronze
silver
golden
metastore
```

Formato de las rutas:

```text
abfss://<container>@adlsssmartdata2698.dfs.core.windows.net/
```

Los seis archivos CSV deben estar disponibles en el contenedor `raw`.

### 3️⃣ Configurar Unity Catalog

Objetos principales:

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

### 4️⃣ Configurar tokens de Databricks

En cada workspace:

```text
User Settings
→ Developer
→ Access tokens
→ Generate new token
```

### 5️⃣ Configurar GitHub Secrets

En GitHub:

```text
Settings
→ Secrets and variables
→ Actions
```

Secrets utilizados por el workflow de despliegue:

| Secret | Descripción |
|---|---|
| `DATABRICKS_ORIGIN_HOST` | URL del workspace de origen |
| `DATABRICKS_ORIGIN_TOKEN` | Token del workspace de origen |
| `DATABRICKS_DEST_HOST` | URL del workspace de destino |
| `DATABRICKS_DEST_TOKEN` | Token del workspace de destino |

Ejemplo de host:

```text
https://adb-xxxxxxxxxxxxxxxx.x.azuredatabricks.net
```

No agregue `/` al final del host.

---

## 💻 Uso

### 🔧 Ejecución manual en Databricks

Ejecute los notebooks en este orden:

```text
1. Enviroment_Prep/1. Enviroment_Preparation
   → Crea catálogo, esquemas, external locations y tablas Delta.

2. Process/2. Ingest_Category_Translation
   → Carga las traducciones de categorías en Bronze.

3. Process/2. Ingest_Customers
   → Carga los clientes en Bronze.

4. Process/2. Ingest_Order_Items
   → Carga los artículos de cada pedido en Bronze.

5. Process/2. Ingest_Orders
   → Carga los pedidos en Bronze.

6. Process/2. Ingest_Payments
   → Carga los pagos en Bronze.

7. Process/2. Ingest_Products
   → Carga los productos en Bronze.

8. Process/3. Transform
   → Limpia, integra y publica la tabla Silver.

9. Process/4. Load
   → Agrega las métricas y publica la tabla Golden.
```

Los notebooks de ingesta pueden ejecutarse en paralelo después de completar la preparación del ambiente.

### 🔄 Despliegue mediante GitHub Actions

El workflow configurado despliega notebooks al workspace de destino cuando se realiza un `push` a `main`.

```bash
git add .
git commit -m "Update Databricks pipeline"
git push origin main
```

Flujo general:

```text
Checkout del repositorio
        ↓
Exportación desde el workspace de origen
        ↓
Codificación del contenido de los notebooks
        ↓
Importación al workspace de destino
        ↓
Limpieza de archivos temporales
```

Workflow:

```text
Dynamic Databricks Notebook Deploy
```

---

## 🔐 Seguridad y control de acceso

El proyecto utiliza tres grupos:

| Grupo | Rol |
|---|---|
| `DEs` | Data Engineers |
| `CTs` | Consultores técnicos |
| `BI` | Analistas BI |

### Data Engineers

Acceso amplio al catálogo, creación y uso de esquemas, lectura de tablas y acceso de lectura/escritura a external locations.

### Consultores técnicos

Acceso de uso al catálogo y esquemas, lectura de las tablas Bronze, Silver y Golden, y lectura de external locations.

### Analistas BI

Acceso limitado al catálogo, al esquema Golden y a la tabla:

```text
catalog_au.golden.olist_sales_summary
```

Notebooks relacionados:

```text
Security/grant
Reversion/revoke
```

---

## ♻️ Reversión

### Revocar permisos

Ejecutar:

```text
Reversion/revoke
```

Este notebook elimina los privilegios asignados a `DEs`, `CTs` y `BI`.

### Eliminar objetos del proyecto

Ejecutar:

```text
Reversion/Drop_tables
```

El notebook contiene instrucciones para eliminar tablas, esquemas, catálogo y external locations. Las instrucciones que eliminan físicamente datos de ADLS permanecen comentadas para evitar borrados accidentales.

---

## 📈 Consultas de validación

### Silver

```sql
SELECT *
FROM catalog_au.silver.olist_orders_enriched
LIMIT 10;
```

### Golden

```sql
SELECT *
FROM catalog_au.golden.olist_sales_summary
LIMIT 10;
```

### Resumen mensual

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

## 🔍 Monitoreo

### En Databricks

- Revisar el resultado de cada celda del notebook.
- Validar los conteos de registros en Bronze, Silver y Golden.
- Consultar el historial de tablas Delta.
- Revisar errores de lectura, escritura o permisos.
- Verificar la ejecución del pipeline desde **Workflows**, cuando se configure como Job.

### En GitHub Actions

- Abrir la pestaña **Actions**.
- Seleccionar `Dynamic Databricks Notebook Deploy`.
- Abrir una ejecución.
- Revisar los logs de exportación e importación de cada notebook.

---

## 🧰 Tecnologías

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

## 👤 Autor

**Daniel Chacón**

Data Engineering | Azure Databricks | PySpark | Delta Lake | Unity Catalog | CI/CD

---

## 📄 Licencia

Agregue un archivo `LICENSE` si el proyecto se distribuirá bajo una licencia específica.

---

**Proyecto:** Ingeniería de datos con Arquitectura Medallion  
**Tecnologías:** Azure Databricks + PySpark + Delta Lake + ADLS Gen2 + Unity Catalog + GitHub Actions
