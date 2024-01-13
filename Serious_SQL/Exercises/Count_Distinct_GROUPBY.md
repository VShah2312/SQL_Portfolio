# Exercise: Record Count & Dsitcint Values

Q: Which actor_id has the most number of unique film_id records in the dvd_rentals.film_actor table?

``` SQL 
SELECT actor_id, count(distinct film_id)
FROM dvd_rentals.film_actor 
GROUP BY actor_id
ORDER BY 2 desc
LIMIT 1;
```

Q: How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?
```SQL
SELECT price, count(distinct fid)
FROM dvd_rentals.nicer_but_slower_film_list
GROUP BY price
ORDER BY 2 desc
LIMIT 3;
``` 
Q: How many unique country_id values exist in the dvd_rentals.city table?
```SQL 
SELECT count(distinct country_id) AS total_countries
FROM dvd_rentals.city;
```

Q: What percentage of overall total_sales does the Sports category make up in the dvd_rentals.sales_by_film_category table?
```SQL
SELECT category, Round(100 * total_sales/ sum(total_sales) OVER(), 2)
FROM dvd_rentals.sales_by_film_category;
```

Q: What percentage of unique fid values are in the Children category in the dvd_rentals.film_list table?
```SQL 
SELECT category, Round( 100* count(distinct fid)/sum(count(distinct fid)) OVER(),2)
FROM dvd_rentals.film_list
GROUP BY category;
```