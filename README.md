# Denormalizing Tables in Data Warehouse: Creating Materialized Views

This project demonstrates how to denormalize tables in a data warehouse by creating materialized views to improve query performance for customer analytics. We use a star schema model with fact and dimension tables and build materialized views for optimized data retrieval. This approach showcases core data engineering skills like schema design, denormalization, and the creation of materialized views for high-performance reporting.

## Table of Contents

- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Schema and Tables](#schema-and-tables)
- [Materialized Views](#materialized-views)
- [Refresh Strategy](#refresh-strategy)
- [Further Improvements](#further-improvements)
- [Conclusion](#conclusion)

## Introduction

In modern data warehousing, normalized tables are often ideal for storage efficiency, but they can lead to suboptimal performance when used for reporting and analytics. By creating materialized views, we can denormalize key data points, pre-compute aggregations, and speed up queries to meet customer reporting requirements.

This project focuses on denormalizing a star schema to create materialized views that help generate summarized insights. The fact table contains billing data, and dimension tables include customer and time-based information. The goal is to build materialized views that aggregate billing information at various levels (by country, year, quarter, category, etc.).

## Project Overview

### Technologies Used

- SQL (PostgreSQL)
- Data Warehousing Concepts
- Star Schema Design
- Materialized Views

### Files in the Repository

1. **`star-schema.sql`**: SQL script to create the schema and tables.
2. **`DimCustomer.sql`**: SQL script for inserting data into the `DimCustomer` table.
3. **`DimMonth.sql`**: SQL script for inserting data into the `DimMonth` table.
4. **`FactBilling.sql`**: SQL script for inserting data into the `FactBilling` table.

### Objectives

- Create a star schema with normalized tables.
- Design materialized views to denormalize the schema for optimized analytics.
- Refresh materialized views periodically for up-to-date reporting.

## Schema and Tables

We have implemented a **star schema** with the following tables:

1. **Fact Table**
   - `FactBilling`: Contains billing information, including customer IDs, month IDs, and billed amounts.

2. **Dimension Tables**
   - `DimCustomer`: Contains customer details, including country and category.
   - `DimMonth`: Contains temporal data such as year, quarter, and month information.

The SQL code for creating these tables is available in the `star-schema.sql` file. Below is a high-level structure of each table:

```sql
-- FactBilling Table
CREATE TABLE "FactBilling" (
    billingid SERIAL PRIMARY KEY,
    customerid INT,
    monthid INT,
    billedamount NUMERIC
);

-- DimCustomer Table
CREATE TABLE "DimCustomer" (
    customerid INT PRIMARY KEY,
    country VARCHAR(100),
    category VARCHAR(100)
);

-- DimMonth Table
CREATE TABLE "DimMonth" (
    monthid INT PRIMARY KEY,
    year INT,
    quarter VARCHAR(10),
    monthname VARCHAR(20)
);
```

## Materialized Views

To support fast data retrieval and efficient reporting, we create materialized views based on common customer requests. These views denormalize the data by pre-aggregating values such as total billed amounts, average amounts, and breakdowns by country, year, and category.

### Example Materialized Views
#### 1. Materialized View: Country Statistics

This view provides the total billed amount per country and year.
```sql
CREATE MATERIALIZED VIEW countrystats (country, year, totalbilledamount) AS
    SELECT country, year, SUM(billedamount)
    FROM "FactBilling"
    LEFT JOIN "DimCustomer" ON "FactBilling".customerid = "DimCustomer".customerid
    LEFT JOIN "DimMonth" ON "FactBilling".monthid = "DimMonth".monthid
    GROUP BY country, year;
```

#### 2. Materialized View: Yearly and Quarterly 
This view summarizes billing by year and quarter, providing total billed amounts.
```sql
CREATE MATERIALIZED VIEW year_quarter_billing AS
    SELECT year, quarter, SUM(billedamount) AS totalbilledamount
    FROM "FactBilling"
    LEFT JOIN "DimCustomer" ON "FactBilling".customerid = "DimCustomer".customerid
    LEFT JOIN "DimMonth" ON "FactBilling".monthid = "DimMonth".monthid
    GROUP BY GROUPING SETS (year, quarter);
```

#### 3. Materialized View: Country and Category Billing

This view aggregates billed amounts by country and customer category, using a rollup for grouping.
```sql
CREATE MATERIALIZED VIEW country_category_billing AS
    SELECT country, category, SUM(billedamount) AS totalbilledamount
    FROM "FactBilling"
    LEFT JOIN "DimCustomer" ON "FactBilling".customerid = "DimCustomer".customerid
    LEFT JOIN "DimMonth" ON "FactBilling".monthid = "DimMonth".monthid
    GROUP BY ROLLUP(country, category)
    ORDER BY country, category;
```

#### 4. Materialized View: Year, Country, and Category Summary

This materialized view uses a cube to create detailed summaries based on year, country, and category.
```sql
CREATE MATERIALIZED VIEW year_country_category_summary AS
    SELECT year, country, category, SUM(billedamount) AS totalbilledamount
    FROM "FactBilling"
    LEFT JOIN "DimCustomer" ON "FactBilling".customerid = "DimCustomer".customerid
    LEFT JOIN "DimMonth" ON "FactBilling".monthid = "DimMonth".monthid
    GROUP BY CUBE (year, country, category);
```

#### 5. Materialized View: Average Bill Amount

This view calculates the average billed amount for each year, quarter, customer category, and country.
```sql
CREATE MATERIALIZED VIEW average_billamount (year, quarter, category, country, average_bill_amount) AS
    SELECT year, quarter, category, country, AVG(billedamount) AS average_bill_amount
    FROM "FactBilling"
    LEFT JOIN "DimCustomer" ON "FactBilling".customerid = "DimCustomer".customerid
    LEFT JOIN "DimMonth" ON "FactBilling".monthid = "DimMonth".monthid
    GROUP BY year, quarter, category, country;
```

## Refresh Strategy

Materialized views need to be kept up-to-date to reflect the most recent data. Depending on the frequency of data changes in the warehouse, we can use a periodic refresh mechanism. In PostgreSQL, you can refresh a materialized view with the following command:

```sql
REFRESH MATERIALIZED VIEW countrystats;
```
This command should be run after significant data changes or based on a schedule (e.g., daily or weekly). It's essential to balance between performance and the need for fresh data, as refreshing materialized views can be resource-intensive.

## Further Improvements

Here are some potential enhancements that can be applied to this project:

* Automated Refresh: Set up a cron job or PostgreSQL trigger to automatically refresh the materialized views at regular intervals.
* Incremental Refresh: If the dataset grows large, you can switch to an incremental refresh strategy to avoid full table scans during the refresh process.
* Partitioning: Consider partitioning the fact table based on time (e.g., year or month) to further optimize the performance of queries and materialized view refreshes.
* Indexing: Create appropriate indexes on the underlying tables and materialized views to further boost query performance.
* Data Compression: Apply data compression techniques on historical data to save storage space without sacrificing read performance.

Conclusion

This project demonstrates a pipeline for denormalizing tables in a star schema data warehouse by creating materialized views. By using SQL queries and materialized views, we optimize reporting and analytics, enabling faster query performance for customer insights. This approach is highly applicable in real-world data engineering scenarios, especially for data warehouses where pre-computation of results leads to enhanced performance.