# Lesson: Window Functions

Window functions are operations or calculations performed on “window frames” or put simply, groups of rows in a dataset.


![Alt text](Window.png)


### Partition By
One of the easiest ways to understand the how the PARTITION BY component of window functions worked was to compare it to a basic GROUP BY query.

![Alt text](Partition1.png)

```SQL 
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sales) AS (
 VALUES
 ('A', 300),
 ('A', 150),
 ('B', 100),
 ('B', 200)
)
SELECT * FROM input_data;

-- Group By Sum
-- Note that the ORDER BY is only for output sorting purposes!
SELECT
  customer_id,
  SUM(sales) AS sum_sales
FROM customer_sales
GROUP BY customer_id
ORDER BY customer_id;

-- Sum Window Function
SELECT
  customer_id,
  sales,
  SUM(sales) OVER (PARTITION BY customer_id) AS sum_sales
FROM customer_sales;
```

Note:In GROUP BY columns are collapsed into one row, were as in PARTITION BY has gives all the rows.  

### Partition By 2 Columns

In the visual example below - you can think of the partitioning as splitting the dataset into smaller groups based on the unique combination of column values from each input to the PARTITION BY

![Alt text](Partition2.png)

```SQL 
-- we remove that existing customer_sales table first!
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sale_id, sales) AS (
 VALUES
 ('A', 1, 300),
 ('A', 1, 150),
 ('A', 2, 100),
 ('B', 3, 200)
)
SELECT * FROM input_data;

-- Sum Window Function with 2 columns in PARTITION BY
SELECT
  customer_id,
  sales,
  SUM(sales) OVER (PARTITION BY customer_id,sale_id) AS sum_sales
FROM customer_sales;
```

### Partition By Multiple Level

We can also use different levels for multiple window functions in a single query - there is one very specific reason why we usually prefer to use window functions to perform multiple level aggregations in this way compared to other methods.

![Alt text](Partition3.png)

```SQL 
SELECT
  customer_id,
  sale_id,
  sales,
  SUM(sales) OVER (PARTITION BY customer_id,sale_id) AS sum_sales,
  SUM(SALES) OVER (PARTITION BY customer_id) AS customer_sales,
  SUM(SALES) OVER () AS total_sales
FROM customer_sales;
```

### Multiple Calculations

We can also apply multiple different window function calculations instead of just using the SUM function like we’ve been using for all the previous examples.

In the following example, we demonstrate how to use AVG and MAX window functions.

![Alt text](Parttiton4.png)

```SQL 
SELECT
  customer_id,
  sale_id,
  sales,
  SUM(sales) OVER (
    PARTITION BY
      customer_id,
      sale_id
  ) AS sum_sales,
  -- the average customer sales is rounded to 2 decimals!
  ROUND(
    AVG(sales) OVER (
      PARTITION BY customer_id
    ),
    2
  ) AS avg_cust_sales,
  MAX(sales) OVER () AS max_sales
FROM customer_sales;
```

Note: The default behaviour for PARTITION BY when the window function has an empty OVER clause is to perform calculations across all the rows of the dataset.


### Ordered Window Functions 
Logically - we can think of the ORDER BY happening after the PARTITION BY clause as the sorting of records will happen within each partition or group that is separated as part of the window function.

![Alt text](Partition5.png)


We can also use a ORDER BY DESC to sort our rows differently and it will provide a reversed outcome!


![Alt text](Partition6.png)

```SQL 
-- we remove any existing customer_sales table first!
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sales_date, sales) AS (
 VALUES
 ('A', '2021-01-01'::DATE, 300),
 ('A', '2021-01-02'::DATE, 150),
 ('B', '2021-01-03'::DATE, 100),
 ('B', '2021-01-02'::DATE, 200)
)
SELECT * FROM input_data;

-- RANK Window Function with default ORDER BY
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    PARTITION BY customer_id
    ORDER BY sales_date
  ) AS sales_date_rank
FROM customer_sales;

-- RANK Window Function with descending ORDER BY
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    PARTITION BY customer_id
    ORDER BY sales_date DESC
  ) AS sales_date_rank
FROM customer_sales;
```

Note:  If there is an empty partition clause - the ORDER BY will still take place - just on all the rows of the dataset as opposed within the separated groups like we would see with a non-empty PARTITION BY clause.

```SQL 
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    ORDER BY sales_date DESC
  ) AS sales_date_rank
FROM customer_sales;
```

### Advanced Window Functions 

#### Bitcoin Mini Case Study: 

