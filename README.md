# retail-etl-tencent-wedata
# Tencent WeData Retail ETL Project Documentation 
## 📈(CPAXTRA intern)

## Project Overview

This project simulates an ETL pipeline built using **Tencent WeData** tools to process retail transaction data (from Online Retail II dataset( https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci )).This documentation aims to show:

* How the tools work
* How we used them to process data
* What the ETL pipeline would look like

---

## Tools Overview

### 1. **DLC (Data Lake Compute)**

* A SQL-based compute engine similar to Databricks SQL.
* Used to run SQL queries for data extraction, cleaning, transformation, and analysis.

### 2. **EMR (Elastic MapReduce)**

* Used for running PySpark or big data jobs.
* You write Spark code in Python and execute on distributed EMR clusters.

### 3. **TCHouse**

* Columnar storage database for analytics.
* Very fast for aggregation, filtering.
* Good target for summary or reporting tables.

### 4. **TDSQL**

* Relational SQL database.
* Used for storing raw transactional data or for integrating with other services.

### 5. **COS (Cloud Object Storage)**

* Where raw files are uploaded before being queried or processed (e.g. CSV).
* DLC and EMR can read from here.

### 6. **Workflow System**

* Visual DAG-based pipeline.
* Nodes can be: SQL, PySpark (EMR), Python Shell, Branch, JDBC, etc.
* Supports dependency, scheduling, and environment variables.

---

## Dataset Description

We used the **Online Retail II** dataset from Kaggle. We uploaded sample rows (\~100) into a TDSQL table:

### Table: `retail.retail_raw_data`

| Column       | Type      |
| ------------ | --------- |
| Invoice      | STRING    |
| StockCode    | STRING    |
| Description  | STRING    |
| Quantity     | INT       |
| InvoiceDate  | TIMESTAMP |
| Price        | DOUBLE    |
| Customer\_ID | STRING    |
| Country      | STRING    |

### Sample Insertion

```sql
INSERT INTO retail.retail_raw_data VALUES
('489444','POST','POSTAGE',1,'2009-12-01 09:55:00',141.0,'12636','USA'),
...
;
```

---

## Workflow Design (ETL)

### Step 1: Extract

* **Node Type**: SQL Node
* **Purpose**: Load data from `retail_raw_data` table.

```sql
SELECT * FROM retail.retail_raw_data;
```

### Step 2: Transform

* **Node Type**: SQL Node
* **Transformations**:

  * Remove rows with NULLs or negative Quantity
  * Calculate `TotalAmount = Quantity * Price`
  * Create `retail_customer_summary` by grouping by `Customer_ID`

```sql
SELECT
  Customer_ID,
  COUNT(DISTINCT Invoice) AS InvoiceCount,
  SUM(Quantity) AS TotalQuantity,
  SUM(Quantity * Price) AS TotalAmount
WHERE Quantity > 0 AND Price > 0
GROUP BY Customer_ID;
```

### Step 3: Load

* **Node Type**: SQL Node
* **Target**: TCHouse table `retail.retail_customer_summary`

```sql
CREATE TABLE IF NOT EXISTS retail.retail_customer_summary (
  Customer_ID STRING,
  InvoiceCount INT,
  TotalQuantity INT,
  TotalAmount DOUBLE
);

INSERT INTO retail.retail_customer_summary
SELECT * FROM {{upstream_transform_node}};
```

---

## Scheduling & Deployment

* Workflow can be scheduled as a daily or hourly task.
* Each node must be submitted before running.
* Check that resource group is configured.

---

## Example Output:

| Customer_ID | InvoiceCount | TotalQuantity | TotalAmount |
|-------------|--------------|----------------|--------------|
| 12636       | 3            | 80             | 189.55       |
| 17519       | 6            | 140            | 255.30       |

## Insight Goals

We can use this pipeline to:
- Understand total spending per customer
- Identify top-performing customers
- Feed data into BI dashboards or customer segmentation model

---
Project by Puttaradol Pongpanich (CPAXTRA Intern) | July 2025  
Mentored by Ankit Kushwaha | For learning purpose only


