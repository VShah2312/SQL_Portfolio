# Lesson: Select and Sort

#### Selecting all columns:
``` SQL
SELECT * 
from schema_name.table _name;  
```

#### Selecting specific column:
``` SQL 
SELECT language_id,name 
FROM dvd_rentals.language;
```

#### LIMIT:
``` SQL
select country 
FROM dvd_rentals.county
LIMIT 10;
```
#### ORDER BY: 
Text columns are sorts alphabetically, where as numeric columns are sorted by value,  default sorting is ascending, use DESC keyword for descending order. Moreover, We can also perform a multi-level sort by specifying 2 or more columns with the ORDER BY clause. Sorting happens in the order of columns mentioned. 

``` SQL 
select country 
FROM dvd_rentals.country 
ORDER by country 
LIMIT 5;
```
## Example: 

Q:  Which category had the lowest total_sales value according to the SALES_BY_FILM_CATEGORY table? What was the total_sales value?

```SQL 
SELECT category, total_sales 
FROM dvd_rentals.sales_by_film_category
ORDER by totaL_sales
LIMIT 1;
```

Q:  What was the latest payment_date of all dvd rentals in the payment table?
``` SQL 
SELECT payment_date
FROM dvd_rentals.payment
ORDER by payment_date DESC
LIMIT 1;
```

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




