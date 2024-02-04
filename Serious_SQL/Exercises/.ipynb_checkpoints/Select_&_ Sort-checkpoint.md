#  Exercise: Select and Sort


Q: What is the name of the category with the highest category_id in the dvd_rentals.category table?
``` SQL 
SELECT name, category_id
FROM dvd_rentals.category
ORDER BY category_id DESC
LIMIT 1;
```

Q: For the films with the longest length, what is the title of the “R” rated film with the lowest replacement_cost in dvd_rentals.film table?
``` SQL 
SELECT length, title, replacement_cost, rating
FROM dvd_rentals.film
WHERE rating= 'R'
ORDER BY length DESC,replacement_cost
LIMIT 1;
```

Q:Who was the manager of the store with the highest total_sales in the dvd_rentals.sales_by_store table?

```sql
SELECT manager, total_sales
FROM dvd_rentals.sales_by_store
ORDER BY total_sales DESC
LIMIT 1;
```

Q: What is the postal_code of the city with the 5th highest city_id in the dvd_rentals.address table?
``` SQL 
SELECT postal_code, city_id
FROM dvd_rentals.address
ORDER BY city_id DESC
LIMIT 5;
```