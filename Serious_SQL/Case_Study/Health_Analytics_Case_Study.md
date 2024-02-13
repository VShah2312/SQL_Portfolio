# Health Analytics Mini Case Study

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the health.user_logs dataset.

The business questions that we need to help the GM answer!

Q: How many unique users exist in the logs dataset?
``` SQL
SELECT count (distinct id)
FROM health.user_logs;
```

Creating a temporary Table: 
```SQL
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS (
SELECT id,
COUNT(*) AS measure_count,
COUNT(DISTINCT measure) as unique_measures
FROM health.user_logs
GROUP BY 1);
``` 

Q: How many total measurements do we have per user on average?
```SQL
SELECT Round(AVG(measure_count)) as average_value
FROM user_measure_count;
```

Q: What about the median number of measurements per user?
``` SQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
```

Q: How many users have 3 or more measurements?
``` SQL
SELECT COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
``` 

Q: How many users have 1,000 or more measurements?
``` SQL
SELECT COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000;
``` 

Q: Have logged blood glucose measurements?
``` SQL
SELECT count(distinct id)
FROM health.user_logs
WHERE measure= 'blood_glucose';
```

Q: Have at least 2 types of measurements?
``` SQL
SELECT count(*)
FROM user_measure_count
WHERE unique_measures >=2;
```


Q: Have all 3 measures - blood glucose, weight and blood pressure?
``` SQL
SELECT count(*)
FROM user_measure_count
WHERE unique_measures =3;
``` 

Q:  What is the median systolic/diastolic blood pressure values?
``` SQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic,
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure= 'blood_pressure'
```
