# Business Questions: 

Q: How many unique users exist in the logs dataset?
``` SQL
SELECT
COUNT DISTINCT user_id
FROM health.user_logs;
```

Creating a temporary Table: 
```SQL
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_cout
SELECT
id,
COUNT(*) AS measure_count,
COUNT(DISTINCT measure) as unique_measures
FROM health.user_logs
GROUP BY 1;
``` 

Q: How many total measurements do we have per user on average?
```SQL
SELECT
ROUND(MEAN(measure_count))
FROM user_measure_count;
```

Q: What about the median number of measurements per user?
``` SQL
SELECT
PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure_count;
```

Q: How many users have 3 or more measurements?
``` SQL
SELECT
  COUNT(*)
FROM user_measure_count
HAVING measure >= 3;
``` 

Q: How many users have 1,000 or more measurements?
``` SQL
SELECT
  SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;
``` 

Q: Have logged blood glucose measurements?
``` SQL
SELECT
  COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';
```

Q: Have at least 2 types of measurements?
``` SQL
SELECT
COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;
```


Q: Have all 3 measures - blood glucose, weight and blood pressure?
``` SQL
SELECT
COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;
``` 

Q:  What is the median systolic/diastolic blood pressure values?
``` SQL
SELECT
PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
```
