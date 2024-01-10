# Serious SQL with Danny

#### Lesson 1:  Notes
1. Use * to select all rows and columns 
2. Use limit to restrict your output for big datasets
3. Order by to sort your results. Default is ASC sort, can use DESC for descending sort. For text column ASC sort alphabetically, for numeric columns its sorts from lowest value to highest. We can sort using more than one column. Sorting happens in the order column is mentioned. We can also mismatch sorting to satify our requirements. 


### Select and Sort 
``` SQL
SELECT column_1, column_2
FROM schema_name.table_name ;
```

###### Selecting all columns:
``` SQL
SELECT * 
from schema_name.table _name;  
```

###### Selecting specific column:
``` SQL 
SELECT language_id,name 
FROM dvd_rentals.language;
```

###### LIMIT 
``` SQL
select country 
FROM dvd_rentals.county
LIMIT 10;
```
Sorting Text column (sorts alphabetically, default sorting is asc)
``` SQL 
select country 
FROM dvd_rentals.country 
ORDER by country 
LIMIT 5;
```
 
Q:  Which category had the lowest total_sales value according to the SALES_BY_FILM_CATEGORY table? What was the total_sales value?

```SQL 
SELECT category, total_sales 
FROM dvd_rentals.sales_by_film_category
ORDER by totaL_sales
LIMIT 1;
```

##### What was the latest payment_date of all dvd rentals in the payment table?
``` SQL 
SELECT payment_date
FROM dvd_rentals.payment
ORDER by payment_date DESC
LIMIT 1;
```

#### Example: 
Q: Which customer_id had the latest rental_date for inventory_id = 1 and 2?

``` SQL 
select customer_id, rental_date, inventory_id
FROM dvd_rentals.rental
ORDER BY inventory_id, rental_date desc
LIMIT 8;
```


Q: In the dvd_rentals.sales_by_film_category table, which category has the highest total_sales?
``` SQL 
select category, total_sales
FROM dvd_rentals.sales_by_film_category
ORDER BY total_sales desc
LIMIT 1
```




