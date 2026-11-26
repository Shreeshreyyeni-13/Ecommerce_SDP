# E-Commerce Sales Data Engineering – Review & Data Quality

## 1. Project Review Questions and Answers

### A. Project Overview and Architecture

**Q1. What is the objective of this project?**

The objective is to build an end-to-end E-Commerce Sales Data Engineering pipeline using Databricks. The pipeline ingests daily order CSV files, stores raw data, performs data cleaning and validation, and generates business-ready datasets for sales analysis.

**Q2. What is Medallion Architecture? Why is it used in this project?**

Medallion Architecture is a data organization approach that divides data processing into three layers:

- Bronze: Stores raw ingested data.
- Silver: Stores cleaned, standardized, and validated data.
- Gold: Stores business-ready data and analytical aggregations.

It helps maintain data quality, data traceability, and a structured transformation process.

**Q3. What technologies are used in this project, and what is their purpose?**

- Databricks: Platform for developing and executing the pipeline.
- PySpark: Performs data cleaning and transformations.
- Lakeflow Spark Declarative Pipelines (SDP): Defines and manages pipeline tables.
- Auto Loader: Incrementally ingests new CSV files.
- Unity Catalog Volume: Stores incoming source files.
- Delta Tables: Stores processed data.

**Q4. What is the source dataset, and what information does it contain?**

The source dataset is `ecommerce_orders.csv`. It contains 11 columns representing order details, customer information, product details, city, category, quantity, unit price, payment mode, and order status.

**Q5. What is the difference between batch processing and streaming processing?**

Batch processing handles a fixed collection of data at scheduled intervals, whereas streaming processing incrementally processes incoming data as it arrives.

In this project, Auto Loader uses Structured Streaming to incrementally discover and ingest new CSV files.

### B. Bronze Layer – Data Ingestion

**Q6. What is the purpose of the Bronze layer?**

The Bronze layer stores raw source data without applying business-level cleaning or transformations. It preserves the original source values and adds metadata columns for tracking and traceability.

**Q7. What is Auto Loader, and why is it used?**

Auto Loader is a Databricks feature used to incrementally discover and process new files from a storage location.

It is used in this project to ingest daily e-commerce CSV files from the Unity Catalog landing folder without manually loading every file.

**Q8. Why are all source columns initially stored as strings in Bronze?**

All source columns are stored as strings to preserve the original input values without performing early type conversions. Data type conversion is handled later in the Silver layer, where invalid values can also be identified.

**Q9. What is the purpose of ingestion_timestamp and source_file_name?**

- `ingestion_timestamp`: Records when the data is ingested into the Bronze layer.
- `source_file_name`: Captures the originating file path.

These columns help track data lineage, identify source files, and investigate ingestion issues.

**Q10. How do you validate whether Bronze ingestion is successful?**

Bronze ingestion is validated by checking:

1. Whether the source CSV file is available.
2. Whether all source records are ingested.
3. Whether all 11 original columns are present.
4. Whether original columns remain strings.
5. Whether ingestion_timestamp and source_file_name are populated.
6. Whether newly arriving files are discovered during subsequent pipeline updates.

**Q11. What is the difference between a source file and a Bronze table?**

The source file is the original CSV containing raw e-commerce order data. The Bronze table stores the ingested records in Delta format along with additional ingestion metadata.

### C. Silver Layer – Data Cleaning and Transformation

**Q12. What is the purpose of the Silver layer?**

The Silver layer transforms raw Bronze data into clean, standardized, and validated records. It handles whitespace removal, text standardization, missing-value normalization, data type conversion, and order amount calculation.

**Q13. What is the difference between silver_orders_cleaned and silver_orders_typed?**

- `silver_orders_cleaned`: Removes unwanted spaces, normalizes repeated whitespace, and standardizes descriptive text fields.
- `silver_orders_typed`: Converts blank strings into NULL values, converts columns into appropriate data types, and calculates order_amount.

These intermediate tables help organize and inspect the transformation process.

**Q14. Why do you use TRIM and REGEXP_REPLACE during data cleaning?**

- TRIM removes unwanted leading and trailing spaces.
- REGEXP_REPLACE normalizes repeated whitespace within text values.

For example, `"  New   Delhi  "` becomes `"New Delhi"`.

**Q15. Why are INITCAP and UPPER used?**

They standardize text values into consistent formats.

Examples:

- `hyderabad` becomes `Hyderabad` using INITCAP.
- `electronics` becomes `Electronics` using INITCAP.
- `upi` becomes `UPI` using UPPER.

This improves consistency during grouping and analysis.

**Q16. Why are blank strings converted into NULL values?**

Blank strings and NULL values are different representations of missing data. Converting blank strings into NULL helps identify missing values consistently and apply validation rules correctly.

**Q17. Why do you use try_cast and try_to_timestamp?**

They attempt data type conversion without failing the transformation because of malformed values. Invalid conversions return NULL, allowing those records to be identified during subsequent validation.

**Q18. What data type conversions are performed in the Silver layer?**

| Column | Silver data type |
|---|---|
| order_date | DATE |
| quantity | INT |
| unit_price | DECIMAL(12,2) |
| order_amount | DECIMAL(18,2) |

The remaining descriptive columns are retained as strings.

**Q19. How is order_amount calculated?**

Order amount is calculated by multiplying quantity by unit_price.

Formula:

`order_amount = quantity * unit_price`

Example:

If quantity = 3 and unit_price = 500:

`order_amount = 3 * 500 = 1500`

The calculated amount is stored as DECIMAL(18,2).

**Q20. What are business validation rules R1–R6?**

These are six rules defined to identify valid and invalid order records.

- R1: order_id must not be NULL or empty.
- R2: order_date must be a valid date.
- R3: quantity must be greater than zero.
- R4: unit_price must be greater than zero.
- R5: customer_id, product_name, category, city, and payment_mode must not be empty.
- R6: order_status must be one of Delivered, Shipped, Pending, Cancelled, or Returned.

**Q21. What happens when a record fails business validation?**

Records that fail one or more validation rules are stored in `silver_orders_rejected` with their rejection reasons. Records that pass the required rules are stored in `silver_orders`.

**Q22. Why is customer_name not included in the mandatory field validation?**

According to the project specification, customer_name is allowed to be empty. Therefore, a record should not be rejected solely because customer_name is missing.

**Q23. Why is duplicate handling important?**

Duplicate handling prevents unintended repeated records from affecting sales calculations and business reporting. However, the actual order grain must be confirmed before deduplication because one order may contain multiple product lines.

### D. Gold Layer – Business Analytics

**Q24. What is the purpose of the Gold layer?**

The Gold layer transforms validated Silver records into business-ready datasets. It provides aggregated information for sales analysis, reporting, and business decision-making.

**Q25. Why is gold_delivered_orders created?**

It acts as a reusable dataset containing only records where order_status is Delivered. The other Gold sales aggregations use this dataset as their source.

**Q26. Why are only Delivered orders considered for revenue analysis?**

The project defines delivered-order revenue using successfully delivered orders only. Pending, Shipped, Cancelled, and Returned orders are excluded to prevent them from inflating delivered-sales revenue.

These records remain available in Silver for order lifecycle analysis.

**Q27. What is a Materialized View, and why is it used?**

A Materialized View stores the result of a query or transformation. It is used in this project to provide precomputed business aggregations for efficient analytical access.

**Q28. What is the purpose of gold_city_sales?**

It groups delivered orders by city and calculates delivered order-record count, total revenue, average order value, and revenue rank. It helps identify city-wise sales performance.

**Q29. What is the purpose of gold_category_sales?**

It groups delivered orders by category and calculates order-record count, total units sold, revenue, average order value, and revenue rank. It helps identify the performance of different product categories.

**Q30. What is the purpose of gold_product_sales?**

It groups delivered records by product and calculates total order records, units sold, total revenue, average selling price, and revenue rank. It helps identify top-performing products.

**Q31. What is the purpose of gold_payment_analysis?**

It groups delivered orders by payment mode and calculates delivered order-record count, total revenue, average transaction value, and revenue rank. It helps understand customer payment preferences and revenue distribution.

**Q32. What is the difference between COUNT(*) and COUNT(DISTINCT order_id)?**

- COUNT(*) counts the total number of records.
- COUNT(DISTINCT order_id) counts the number of unique order IDs.

If one order contains three product lines, COUNT(*) returns 3, whereas COUNT(DISTINCT order_id) returns 1.

The correct approach depends on the source data grain and the business definition of an order.

**Q33. How do you validate Gold revenue calculations?**

Revenue is validated by comparing the sum of total_revenue across city, category, product, and payment aggregations with the total order_amount in gold_delivered_orders.

The totals should match when every delivered record has populated grouping dimensions and is included in each aggregation.

**Q34. What is the difference between gold_order_summary and gold_sales_summary?**

- `gold_order_summary`: Planned dataset to report order lifecycle and status distribution across all valid Silver records.
- `gold_sales_summary`: Planned dataset to report overall business KPIs and delivered-sales performance.

Both are pending implementation in the current project documentation.





The E-Commerce Sales Data Engineering pipeline follows Medallion Architecture to ensure structured data ingestion, cleaning, validation, and business-level analysis.

The Bronze layer preserves raw data, the Silver layer ensures data quality, and the Gold layer delivers analytics-ready datasets for business insights.
