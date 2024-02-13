# Lesson: Record Count & Dsitcint Values

### Record Counts & Distinct Values
We can use COUNT(*) count total records in a table. Keep in mind it doesnt check for nulls. We use Alias to rename expression. 

Aliases can also be used to name other SQL expressions such as database tables, subqueries and common table expressions (CTEs) as they vastly improve SQL code readability.

We can use the DISTINCT keyword to obtain unique values from a deduplicated target column.

We can use the COUNT function with the DISTINCT keyword to find the number of unique values of a specific column.


#### GROUP BY 
We can use GROUP BY clause with a aggregate function to help us generate a basic frequency value counts output. 

One way to think of the GROUP BY is to imagine our dataset being divided into different groups based off the values of selected columns.

1. The first step would be to evaluate the different values within the column expression we’re using for the grouping element used for the GROUP BY clause.
2. nce we have split the dataset into the groups specified for the GROUP BY we then apply the aggregate function within each of these grouped datasets to condense our output to a single row from each group.
3. Mathematical aggregate functions to GROUP BY example such as SUM, MEAN, STDDEV, MAX and MIN
4. Once the calculations are complete for each separate group, the condensed 1 row from each group is then combined together to form the final output for the original SQL statement with the GROUP BY clause. 
5. When we use GROUP BY on 2+ columns, the subsequent COUNT function will aggregate the records based off the unique combination of values in these columns instead of just a single 1.

 (Only 1 row is returned for each group. In any query while selecting, non aggregate columns needs to go in a GROUP BY clause) 

We often sort output when analyzing data to see which values dominate the frequency count using ORDER BY clause.

Take special attention that any ORDER BY clause must occur after the GROUP BY


#### Adding Percentage Column

Sometimes the frequency is just not enough to really understand the frequency at a quick glance, so we like to create an additional percentage column to our dataset.

This percentage decimal seems a bit ugly and annoying to look at it so to further spice it up - we can also multiple this percentage value by 100 and round it to 1 decimal place using a ROUND function.
``` SQL 
SELECT
  rating, 
  COUNT(*) AS frequency,
  COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER() AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
``` 

``` SQL 
SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
``` 

## Example: 

Q: How many rows are there in the film_list table?

```SQL 
SELECT count(*) as row_count
FROM dvd_rentals.film_list;
```

Q: What are the unique values for the rating column in the film table?

```SQL 
SELECT distinct rating 
FROM dvd_rentals.film_list;
```

Q: How many unique category values are there in the film_list table?

```SQL 
SELECT count(distinct category)
FROM dvd_rentals.film_list;
```

Q: What is the frequency of values in the rating column in the film_list table?

``` SQL 
SELECT rating, count(*) as Frequency
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY 2 DESC;
```

Q: What are the 5 most frequent rating and category combinations in the film_list table?

```SQL 
SELECT rating, category, count(*) as frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY 3 DESC
LIMIT 5; 
```