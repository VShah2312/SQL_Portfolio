# Lesson: Identifying Duplicate Records

### Identifying Duplicates
Duplicate records are literally everywhere when you start dealing in the world of real-life messy datasets.

In this  lesson we will be using health.user_logs.

### Introduction to Health Data 

At first glance: 
- We observe that we have 43891 records. 
```SQL 
SELECT count(*)
FROM health.user_logs;
```

- We have 554 unique id's. 
```SQL 
SELECT count(*)
FROM health.user_logs;
```

- Lets inspect the measure column and take a look at the most frequent values within this column.
```SQL 
SELECT measure, count(*) AS frequency,
Round(100* count(*)/sum(count(*)) OVER(),2) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY 2 DESC; 
```
- Lets inspect the id column and take a look at the most frequent values within this column.
```SQL 
SELECT id, count(*) AS frequency,
Round(100* count(*)/sum(count(*)) OVER(),2) AS percentage
FROM health.user_logs
GROUP BY id
ORDER BY 2 DESC; 
```

- Lets inspect individual column distributions: 
    - Measure Column: 
    ```SQL 
    SELECT measure_value,
    count(*) as frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 desc
    LIMIT 10;
    ```
    So we notice many 0 values for measure_value filed.

    Digging deeper we observe that most of those measure_value = 0 are occuring when measure = 'blood_pressure'.

    Moreover, when the blood pressure is measured - sometimes the measure_value field is empty and the relevant systolic and diastolic fields are populated with the valid records!

    But what happened to the records where measure = 'blood_pressure' but the measure_value != 0 ? 

    It looks like that systolic value is sometimes being used at that measure_value and sometimes the measure_value is 0!


    - Systolic Column: 
    ```SQL 
    SELECT systolic,
    count(*) as frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 desc
    LIMIT 10;
    ```

    For systolic column, we observe null values.

    On further inspection, we observe that systolic only has non-null records when measure = 'blood_pressure'.

    - Diastolic Column: 
    ```SQL 
    SELECT diastolic,
    count(*) as frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 desc
    LIMIT 10;
    ```

      For diastolic column, we observe null values.

    And like systolic, we observe that diastolic only has non-null records when measure = 'blood_pressure'.



## How to deal with duplicates: 

In SQL there are a few different ways to deal with these duplicate records:

- Remove them in a SELECT statement
- Recreating a “clean” version of our dataset
- Identify exactly which rows are duplicated for further investigation or
- Simply ignore the duplicates and leave the dataset alone


Coming back to dataset on our hand, lets check if we even have duplicate records in our table!

#### Detecting Duplicate Records: 

1. #####  Subqueries: 
    A subquery is essentially a query within a query.

    ###### Example: 
    ```SQL 
    SELECT COUNT(*)
    FROM (
        SELECT DISTINCT *
        FROM health.user_logs
    ) AS subquery;
    ```

2. ##### CTE (Common Table Expression): 
    A CTE is a SQL query that manipulates existing data and stores the data outputs as a new reference. Subsequent CTEs can refer to existing datasets, as well as previously generated CTEs. This allows for quite complex nested queries and operations to be performed, whilst keeping the code nice and readable!

    ##### Example: 
    ```SQL 
    WITH deduped_logs AS (
    SELECT DISTINCT *
    FROM health.user_logs
    )

    SELECT COUNT(*)
    FROM deduped_logs;
    ```

3. ##### Temporary Tables: 
    This is a very common approach when you know that you will only be analyzing the deduplicated dataset, and you will ignore the original one with duplicates. 

    The main benefit of using temporary tables is removing the need to always run the same DISTINCT command everytime you want to run a query on the deduplicated records.

    Temporary tables can also be used with indexes and partitions to speed up performance of our SQL queries.

    ##### Example: 
    ```SQL
    DROP TABLE IF EXISTS deduplicated_user_logs; 

    CREATE TEMP TABLE deduplicated_user_logs AS
    SELECT DISTINCT *
    FROM health.user_logs;
    ```

#### Which method is best for us to use in this case?

#### We will answer this question by asking "Will I need to use the deduplicated data later?"

- If yes - opt for temporary tables.

- If no - CTEs are your friend.



#### Comparing Counts: 

We now have the row counts of the original table 43,891 and of our deduplicated table 31,004

It’s pretty safe to say that we have some deuplicate records!

By comparing the counts of the original and the deduplicated, we can prove the presence of duplicates.


#### Identifying Duplicates: 
```SQL 
SELECT id, log_date, measure, measure_value, systolic, diastolic,COUNT(*) AS frequency
FROM health.user_logs
GROUP BY id,log_date,measure,measure_value, systolic, diastolic
ORDER BY frequency DESC;
```

Notice how the frequency for some of these values is 1 - whilst some are greater than 1?

This is exactly how we know which unique combinations of the columns have duplicates!

So if we only want duplicate records to be returned , we can add filter that count(*) value to be greater than 1. And we can get deduplicated data by setting count() value to be equal to 1. 

##### Note that we cannot use the same * within the GROUP BY grouping elements as it will throw a syntax error.


#### Ignoring Duplicate Values: 

For our example, multiple measurements can be taken on the same day at different times, but you may notice this information is missing as the log_date column does not show timestamp values! That part about multiple measurements could be critical in the approach we wish to take with these duplicates.

 Thus, we must always think critically about the data that we have and what it really represents in terms of actual behaviour or processes that create them!