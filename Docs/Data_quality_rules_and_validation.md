# Data Quality Rules – E-Commerce Sales Data Engineering

Data quality rules are applied in the Silver layer to ensure that e-commerce order records are complete, valid, and consistent before they are used for business analytics.

## 1. Business Validation Rules (R1–R6)

| Rule ID | Column | Validation Rule | Expected Condition |
|---|---|---|---|
| R1 | order_id | Order ID Validation | Must not be NULL or empty. |
| R2 | order_date | Date Validation | Must be a valid date. |
| R3 | quantity | Quantity Validation | Must be greater than zero. |
| R4 | unit_price | Price Validation | Must be greater than zero. |
| R5 | Required Fields | Mandatory Field Validation | customer_id, product_name, category, city, and payment_mode must not be empty. |
| R6 | order_status | Status Validation | Must be Delivered, Shipped, Pending, Cancelled, or Returned. |

**Note:** customer_name is allowed to be empty according to the project specification.

## 2. Data Cleaning and Standardization Rules

| Column | Data Quality Rule | Expected Result |
|---|---|---|
| order_id | Remove leading and trailing spaces. | Standardized order ID. |
| customer_id | Remove unwanted spaces. | Clean customer ID. |
| customer_name | Apply TRIM and INITCAP. | Standardized customer name. |
| city | Apply TRIM and INITCAP. | Consistent city names. |
| product_name | Apply TRIM and INITCAP. | Standardized product names. |
| category | Apply TRIM and INITCAP. | Consistent category names. |
| payment_mode | Apply TRIM and UPPER. | Standardized payment modes. |
| order_status | Apply TRIM and INITCAP. | Consistent order statuses. |
| order_date | Convert to DATE. | Valid date format. |
| quantity | Convert to INT. | Integer values. |
| unit_price | Convert to DECIMAL(12,2). | Standardized price values. |
| order_amount | Calculate quantity × unit_price. | Accurate order amount. |

## 3. Data Quality Dimensions

| Dimension | Description | Application |
|---|---|---|
| Completeness | Ensures required fields contain values. | Mandatory field validation. |
| Validity | Ensures values follow defined rules. | Date, quantity, price, and status validation. |
| Consistency | Ensures uniform data formats. | Text cleaning and standardization. |
| Accuracy | Ensures calculated values are correct. | Order amount verification. |
| Uniqueness | Identifies duplicate records. | Duplicate identification based on business grain. |
| Traceability | Tracks the origin of records. | ingestion_timestamp and source_file_name. |

## 4. Expected Data Quality Outcome

- All mandatory fields should contain valid values.
- Order dates should be valid.
- Quantity and unit price should be greater than zero.
- Order status should match the accepted values.
- Text fields should follow consistent formats.
- Order amount should be calculated correctly.
- Duplicate records should be identified after confirming the order grain.
- Invalid records should be stored in `silver_orders_rejected` with rejection reasons.
- Valid records should be stored in `silver_orders`.

**Conclusion:** These data quality rules ensure that the Silver layer contains clean, standardized, and validated e-commerce order data suitable for downstream Gold-layer analytics.
