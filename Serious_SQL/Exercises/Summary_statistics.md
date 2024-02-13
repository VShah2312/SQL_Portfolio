# Exercise: Summary Statistics

Q: What is the average, median and mode values of blood glucose values to 2 decimal places?

```SQL 
SELECT measure,
  Round(AVG(measure_value),2) as Average,
  Round(Cast(Percentile_Cont(0.5) WITHIN GROUP (ORDER BY measure_value)AS Numeric) ,2) as Median, 
  Round(Mode() WITHIN GROUP (ORDER BY measure_value),2) as Mode
FROM health.user_logs
WHERE measure='blood_glucose'
GROUP BY measure;
```

Q: What is the most frequently occuring measure_value value for all blood glucose measurements?

```SQL 
SELECT measure_value, count(*) as Frequency
FROM health.user_logs
WHERE measure='blood_glucose'
GROUP BY measure_value
ORDER BY frequency DESC
LIMIT 1;
```

Q: Calculate the 2 Pearson Coefficient of Skewness for blood glucose measures given the following formulas:

```SQL 
With stat_summary AS(
SELECT measure,
  Round(AVG(measure_value),2) as Average,
  Round(Cast(Percentile_Cont(0.5) WITHIN GROUP (ORDER BY measure_value)AS Numeric) ,2) as Median, 
  Round(Mode() WITHIN GROUP (ORDER BY measure_value),2) as Mode,
  Round(STDDEV(measure_value),2) as Std
FROM health.user_logs
WHERE measure='blood_glucose'
GROUP BY measure)

SELECT 
  Round((Average-Mode)/ Std,2) as Coeff1,
  Round(3 * ((Average-Median)/ Std) ,2) as Coeff2
FROM stat_summary; 
```