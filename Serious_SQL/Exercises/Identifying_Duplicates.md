# Exercise : Identifying Duplicate Records

Q: Which id value has the most number of duplicate records in the health.user_logs table?

```SQL 
WITH groupby_counts AS (
SELECT id, log_date, measure, measure_value, systolic, diastolic,COUNT(*) AS frequency
FROM health.user_logs
GROUP BY id,log_date,measure,measure_value, systolic, diastolic)


SELECT id, SUM(frequency) AS total_duplicate_rows
FROM groupby_counts
WHERE frequency >1
GROUP BY id
ORDER BY 2 DESC
LIMIT 1; 
```

Q; Which log_date value had the most duplicate records after removing the max duplicate id value from question 1?

```SQL 
WITH groupby_counts AS (
SELECT id, log_date, measure, measure_value, systolic, diastolic,COUNT(*) AS frequency
FROM health.user_logs
WHERE id != '054250c692e07a9fa9e62e345231df4b54ff435d'
GROUP BY id,log_date,measure,measure_value, systolic, diastolic )


SELECT log_date, SUM(frequency) AS total_duplicate_rows
FROM groupby_counts
WHERE frequency >1
GROUP BY log_date
ORDER BY 2 DESC
LIMIT 1; 
```

Q: Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?

```SQL 
SELECT measure_value, count(*) as frequency
FROM health.user_logs
WHERE measure = 'weight'
GROUP BY measure_value
ORDER BY 2 DESC
LIMIT 1; 
```

Q: How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?

```SQL 
WITH groupby_counts AS (
SELECT id, log_date, measure, measure_value, systolic, diastolic,COUNT(*) AS frequency
FROM health.user_logs
WHERE measure= 'blood_pressure'
GROUP BY id,log_date,measure,measure_value, systolic, diastolic)


SELECT count(*) as single_duplicate_rows, sum(frequency) as total_duplicate_rows
FROM groupby_counts
WHERE frequency >1; 
```

Q: What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?

```SQL 
WITH all_measure_values AS 
( SELECT measure_value, count(*) as total_records, sum(count(*)) OVER() as overall_total
FROM health.user_logs
WHERE measure='blood_pressure'
GROUP BY 1)

SELECT measure_value, total_records, overall_total, Round(100 * total_records/ overall_total, 2) as Percentage
FROM all_measure_values 
WHERE measure_value= 0;
```

Q: What percentage of records are duplicates in the health.user_logs table?

```SQL 
WITH groupby_counts AS (
SELECT id, log_date, measure, measure_value, systolic, diastolic,COUNT(*) AS frequency
FROM health.user_logs
GROUP BY id,log_date,measure,measure_value, systolic, diastolic)

SELECT Round(100* SUM(CASE WHEN frequency > 1 THEN frequency -1 ELSE 0 END)/ SUM(frequency), 2) 
FROM groupby_counts;
```