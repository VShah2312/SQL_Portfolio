# Lesson: Distribution Functions

Recall: Summary Statistics from last lesson. 

```SQL 
SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value,
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    -- this function actually returns a float which is incompatible with ROUND!
    -- we use this cast function to convert the output type to NUMERIC
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';
```
| Column Name    | Data Value |
| -------- | ------- |
| minimum_value  | 0.00    |
| maximum_value | 39642120.00    |
| mean_value    | 28786.85    |
| median_value    | 75.98   |
| mode_value    | 68.49    |
| standard_deviation    | 1062759.55    |
| variance_value    | 1129457862383.41    |


A few questions come to mind straightaway:

* Does it make sense to have such low minimum values and such a high value?
* Why is the average value 28,786kg but the median is 75.98kg?
* The standard deviation of values is WAY too large at 1,062,759kg

This is when cumulative distrubtion function comes to rescue. 


### Cumulative Distribution Function: 

In mathematical terms, a cumulative distribution function takes a value and returns us the percentile or in other words: the probability of any value between the minimum value of our dataset X and the value V as shown below:
$$
F(V)=∫Vmin(X)f(x)dx=Pr[min(X)≤x≤V]
$$

The beautiful thing about probabilities is that they always add up to 1!

### Calculating Percentile 

In order to calculate percentile: 

1. Order all of the weight measurement values from smallest to largest
2. Split them into 100 equal buckets - and assign a number from 1 through to 100 for each bucket. 

We can actually achieve both of these algorithmic steps in a single bit of SQL functionality with the all-powerful analytical function

Firstly the OVER and ORDER BY clauses in the following query help us re-order the dataset by the measure_value column - it sorts by ascending order by default)

Then the NTILE window function is used to perform the assignment of numbers 1 through 100 for each row in the records for each measure_value. 

```SQL 
SELECT measure_value,
NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight'
```

3. For each bucket:
- calculate the minimum value and the maximum value for the ceiling and floor values
- count how many records there are

Since we now have our percentile values and our dataset is split into 100 buckets - we can simply use a GROUP BY on the percentile column from the previous table to calculate our MIN and MAX measure_value ranges for each bucket and also the COUNT of records for the percentile_counts field.

We can also use the previous query in a CTE so we can pull all the calculations in a single SQL query:

```SQL 
WITH percentile_values AS (SELECT measure_value,
NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT percentile, min(measure_value) as floor_value, 
max(measure_value) as ceiling_value,
count(*) as percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile
```

The first place to take a careful look at is the tails of the values, namely percentile = 1 and percentile = 100. 

So from first glance you get the following insights right away:

- 28 values lie between 0 and ~29KG
- 27 values lie between 136.53KG and 39,642,120KG

When we think of those small values in the 1st percentile under 29kg, a few things should come to mind:

1. Maybe there were some incorrectly measured values leading to some 0kg weight measurements
2. Perhaps some of the low weights under 29kg were actually valid measurements from young children who were part of the customer base

For the 100th percentile we could consider:

1. Does that 136KG floor value make sense?
2. How many error values were there actually in the dataset?


### Deep Dive Into 100th Percentile

Lets begin by looking into some more window functions. 

```SQL 
WITH percentile_values AS (SELECT measure_value,
NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT measure_value,
-- these are examples of window functions below
ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
RANK() OVER (ORDER BY measure_value DESC) as rank_order,
DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;
```
Q: What is the difference between ROW_NUMBER, RANK and DENSE_RANK window functions?

* The ROW_NUMBER() function is self-explanatory, as you’ve already seen the data. It simply assigns a consecutive ranking to each row ordered by measure_value. If two rows have the same value, they won’t have the same ranking.

* The RANK() function creates a ranking of the rows based on the provided columns. It starts with assigning “1” to the first row in the order and then gives higher numbers to rows lower in the order. If rows have the same value, they’re ranked the same. However, the next spot is shifted accordingly. For example, if two rows are 2th (have the same rank), the next row will be 4th (i.e., 3rd doesn’t exist).

* The DENSE_RANK() function is rather similar. The only difference is that it doesn’t leave gaps in the ranking values. Even though more than one row can have the same rank, the rank of the next row will be one plus the previous number. For example, if two rows are 2rd, the next row will be 3rd.


### Large Outliers
So it looks like there are at least 3 HUGE values which are outliers in our weight values in that top percentile.


| measure_value| row_number_order| rank_order | dense_rank_order |
| -------- | ------- | -------- | ------- |
| 39642120 | 1 | 1 | 1 |
|39642120	| 2	| 1	| 1|
|576484	|3	|3	|2|
|200.487664	|4	|4	|3|


The last 200kg value MIGHT be a real measurement - so we would have to apply our judgement to determine whether this is actually an outlier or not.

Now that we’ve identified these large outliers - the next question is: what do we do with them?

The answer is luckily simple: you remove them!

### Small Outliers


```SQL 
WITH percentile_values AS (SELECT measure_value,
NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT measure_value,
-- these are examples of window functions below
ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
RANK() OVER (ORDER BY measure_value DESC) as rank_order,
DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
FROM percentile_values
WHERE percentile = 1
ORDER BY measure_value;
```
it seems like there are a few 0 weight values as well as a few VERY small values.

Again, this will be a judgement call as to whether we should keep or remove some of these values.

I would just remove the 0 values and keep everything else.


### Removing Outliers
Let’s create a temporary clean_weight_logs table with our outliers removed by applying strict inequality conditions.

1. Create a temporary table using a CREATE TEMP TABLE <> AS statement

```SQL 
DROP TABLE IF EXISTS clean_weight_logs;
CREATE TEMP TABLE clean_weight_logs AS (
SELECT *
FROM health.user_logs
WHERE measure = 'weight'
AND measure_value > 0
AND measure_value < 201);
```
2. Calculate summary statistics on this new temp table
```SQL 
SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value,
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
  -- this function actually returns a float which is incompatible with ROUND!
  -- we use this cast function to convert the output type to NUMERIC
  CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
  2) AS median_value,
  ROUND(MODE() WITHIN GROUP (ORDER BY measure_value),2) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM clean_weight_logs;
```

3. Show the new cumulative distribution function with treated data
```SQL 
WITH percentile_values AS (SELECT measure_value,
NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM clean_weight_logs)

SELECT percentile,
MIN(measure_value) AS floor_value,
MAX(measure_value) AS ceiling_value,
COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;
```

We have a WIDTH_BUCKET function that helps us split the range of values into a number of buckets we would like.

```SQL 
SELECT
WIDTH_BUCKET(measure_value, 0, 200, 50) AS bucket,
AVG(measure_value) AS measure_value,
COUNT(*) AS frequency
FROM clean_weight_logs
GROUP BY bucket
ORDER BY bucket;
```

Visual Representation of our clean dataset: 

![Alt text](<Screenshot 2024-01-19 at 11.42.55 AM.png>)