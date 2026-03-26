# 📊 Data Engineering Mini Project Documentation

## Assumptions & Data Quality Rules

---

# 🏗️ 1. Architecture Overview

The data pipeline follows a **Medallion Architecture**:

* **Bronze Layer** → Raw ingestion from source
* **Silver Layer** → Data cleaning & standardization
* **Gold Layer** → Business-ready data (DIM, FACT, KPIs)

---

# 🥉 2. Bronze Layer (Raw Ingestion)

### 📁 File: `bronze/ingest_bronze.ipynb`

## ✅ Assumptions

1. Source data is available in `azure_blob_storage` catalog
2. All tables are ingested **as-is without transformation**
3. Schema is assumed to be **consistent at source**
4. No filtering, no validation at this stage

---

## ⚙️ Logic Implemented

* Dynamically loads all tables:

  ```python
  tables = [t.name for t in spark.catalog.listTables("azure_blob_storage")]
  ```
* Adds ingestion timestamp:

  ```python
  df = df.withColumn("ingestion_ts", current_timestamp())
  ```
* Stores as Delta tables:

  ```
  workspace.bronze.bronze_<table_name>
  ```

---

## 🧪 Data Quality Rules (Bronze)

✔ No validation applied
✔ Raw data preserved for audit
✔ Schema drift tolerated
✔ Historical trace maintained via `ingestion_ts`

---

# 🥈 3. Silver Layer (Clean & Standardized Data)

## ✅ Assumptions

1. Data may contain:

   * Null values
   * Incorrect formats (e.g., dates as strings)
   * Duplicates
2. All transformations happen here (NOT in Bronze or Gold)
3. Data is standardized before analytics

---

## ⚙️ Key Transformations

### 🔹 1. Date Standardization

* Convert string → date format:

  ```python
  to_date(col("order_date"), "dd-MM-yyyy")
  ```

---

### 🔹 2. Null Handling

* Remove or filter invalid rows where critical fields are null:

  * `order_id`
  * `customer_id`
  * `product_id`

---

### 🔹 3. Deduplication

* Remove duplicate records using primary keys

---

### 🔹 4. Data Normalization

* Standardize text fields:

  * Lowercase emails
  * Proper case names

---

### 🔹 5. Exchange Rate Preparation

* Ensure:

  * `currency` is valid
  * `rate_to_usd` exists for each date

---

## 🧪 Data Quality Rules (Silver)

✔ No nulls in primary keys
✔ Dates are properly formatted
✔ No duplicate records
✔ Currency and exchange rates validated
✔ Invalid records removed or filtered

---

# 🥇 4. Gold Layer (Business Layer)

## ✅ Assumptions

1. Data is clean and analytics-ready
2. Business logic is applied only here
3. Currency normalization is required

---

## ⭐ Data Modeling

### 📌 Dimension Tables

* `dim_customer`
* `dim_product`
* `dim_date`

### 📌 Fact Table

* `fact_orders`

---

## ⚙️ Key Transformations

### 🔹 1. Currency Conversion (USD Standardization)

```python
revenue_usd = quantity * price * rate_to_usd
```

✔ Ensures consistent global reporting
✔ Applied in **fact table only**

---

### 🔹 2. Status Handling

* All statuses retained:

  * pending
  * completed
  * shipped
  * cancelled

✔ No filtering in fact table
✔ KPI layer controls business logic

---

### 🔹 3. Country Attribution

* Revenue is calculated using:

  ```text
  orders.country
  ```

✔ Represents transaction location
✔ Not based on customer or product country

---

## 🧪 Data Quality Rules (Gold)

✔ Revenue must be > 0
✔ Foreign keys must exist
✔ Exchange rate must be available
✔ All joins must be valid
✔ No nulls in critical analytical fields

---

# 📊 5. KPI Layer Assumptions

## 🔹 Revenue Calculation

* Only **completed orders** are considered:

  ```text
  Revenue = SUM(revenue_usd WHERE status = 'completed')
  ```

---

## 🔹 Customer Acquisition

* Based on:

  ```text
  First order date of each customer
  ```

---

## 🔹 Data Quality Score

* Defined as:

  ```text
  Valid Records / Total Records
  ```

Where valid records satisfy:

* No null keys
* Valid revenue
* Proper joins

---

# ⚠️ 6. Key Design Decisions

### ✔ Separation of Layers

* Bronze → Raw
* Silver → Clean
* Gold → Business

---

### ✔ Currency Handling in Gold

* Avoids repeated computation
* Ensures consistent KPIs

---

### ✔ Fact Table Stores All Data

* No filtering applied
* Maintains auditability

---

### ✔ KPI Layer Controls Business Logic

* Flexible
* Scalable
* Easy to modify

---

# 🏆 Conclusion

This pipeline ensures:

✔ Data reliability
✔ Scalability
✔ Business accuracy
✔ Clear separation of concerns

The design follows **industry-standard medallion architecture** and supports efficient analytical querying and reporting.
