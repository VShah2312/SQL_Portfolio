# Marketing Analytics Case Study 

## Case study overview: 

Personalized customer emails based off marketing analytics is a winning formula for many digital companies, and this is exactly the initiative that the leadership team at DVD Rental Co has decided to tackle!

We have been asked to support the customer analytics team at DVD Rental Co who have been tasked with generating the necessary data points required to populate specific parts of this first-ever customer email campaign.

Throughout this marketing case study we will cover many SQL data manipulation and analysis techniques. The aim is to further extend your SQL knowledge base and also expose you to some scenarios where you can apply some neat tricks that I’ve picked up over the years!

![Alt text](Unknown-1.png)

### Key Business Requirements

The marketing team have shared with us requirements of the email they wish to send to their customers.

#### Requirement #1: Top 2 Categories

For each customer, we need to identify the top 2 categories for each customer based off their past rental history. These top categories will drive marketing creative images as seen in the travel and sci-fi examples in the draft email.

#### Requirement #2: Category Film Recommendations 

The marketing team has also requested for the 3 most popular films for each customer’s top 2 categories.

There is a catch though - we cannot recommend a film which the customer has already viewed.

If there are less than 3 films available - marketing is happy to show at least 1 film.

Any customer which do not have any film recommendations for either category must be flagged out so the marketing team can exclude from the email campaign - this is of high importance!

#### Requirement #3 & #4: Individual Customer Insights

The number of films watched by each customer in their top 2 categories is required as well as some specific insights.

For the 1st category, the marketing requires the following insights (requirement 3):

1. How many total films have they watched in their top category?
2. How many more films has the customer watched compared to the average DVD Rental Co customer?
3. How does the customer rank in terms of the top X% compared to all other customers in this film category?

For the second ranking category (requirement 4):

1. How many total films has the customer watched in this category?
2. What proportion of each customer’s total films watched does this count make?

Note the specific rounding of the percentages with 0 decimal places!

#### Requirement 5: Favorite Actor Recommendations

Along with the top 2 categories, marketing has also requested top actor film recommendations where up to 3 more films are included in the recommendations list as well as the count of films by the top actor.

We have been given guidance by marketing to choose the actors in alphabetical order should there be any ties - i.e. if the customer has seen 5 Brad Pitt films vs 5 George Clooney films - Brad Pitt will be chosen instead of George Clooney.

The same logical business rules apply - in addition any films that have already been recommended in the top 2 categories must not be included as actor recommendations.

If the customer doesn’t have at least 1 film recommendation - they also need to be flagged with a separate actor exclusion flag.


# Data Exploration: 

The Analytics team at DVD Rental Co have provided an entity-relationship diagram (also known as an “ERD”) to highlight which tables in the dvd_rentals schema we should focus on.

![Alt text](Unknown.png)

Entity-Relationship Diagrams are super popular in the workplace and you will most definitely come across them many many times!

There are a few things to note about ERDs:

- all column names as well as the data type are shown for each table
- table indexes and foreign keys are usually highlighted to show the linkages between tables

#### Table 1: Rental 

This dataset consists of rental data points at a customer level. There is a unique sequential rental_id for each record in the table which corresponds to an individual customer_id purchasing or renting a specific item with a inventory_id. There is also information about the rental_date and return_date as well as which staff member served the customers.

The last_update field is an internal database timestamp for when the datapoints were inserted into the table.

In the ERD - we can see that there is a linkage between this rental table with the inventory table via the inventory_id field.


#### Table 2 Inventory: 

This inventory dataset consists of the relationship between specific items available for rental at each store - note that there may be multiple inventory items for a specific film at a unique store.

In other words - say a specific film like “Harry Potter & The Chamber of Secrets” might have 4 copies at store #1 and an additional 4 copies in store #2 - each record in this dataset will have a separate inventory_id whilst the film_id will all be the same and the store_id changes according to the store number.

This inventory table is linked to the previous rental table via the inventory_id which is an integer datatype.

#### Table 3 Film: 

The third table in our ERD is the film table which helps us identify the title of films rented by DVD Rental Co customers. The film title, description as well as special features and other fields are available for further analysis.

One specific column which might be of interest is the rental_rate column which represents the cost of each rental - something which could be useful if we wanted to look at the sales performance of DVD Rental Co.

#### Table 4 Film_Category: 
The 4th table in the ERD is film_category which shows the relationship between film_id and category_id.

Multiple films will appear in each relevant category_id which will map one-to-one with our next table in our ERD, the category table.

#### Table 5 Category: 

The 5th table in the ERD is the category table which simply contains a one to one mapping between category_id and the name of each category.

#### Table 6 Film_Actor: 
The 6th table in our ERD is film_actor which shows actors who appeared in each film based off related actor_id and film_id values.

An actor can appear in multiple films and a film can feature multiple actors. This relationship between values is what we usually call a “many-to-many” relationship. 

#### Table 7 Actor: 
The actor table simply shows the first and last name for each actor based off their unique actor_id - we can trace which films a specific actor appears in by joining this table onto the previously discussed film_actor table on the actor_id column.

#### The Final State: 

The key columns that we will need to generate include the following data points at a customer_id level:

* category_name: The name of the top 2 ranking categories
* rental_count: How many total films have they watched in this category
* average_comparison: How many more films has the customer watched compared to the average DVD Rental Co customer
* percentile: How does the customer rank in terms of the top X% compared to all other customers in this film category?
* category_percentage: What proportion of total films watched does this category make up?


## Scripting Solution: 

#### P - Problem
What is the business problem and what sort of value can we generate by solving it?

#### E - Exploration
What available resources do we have and what initial data exploration can we perform to better understand our data?

Here is a sample of the analysis we completed for our exploration of the dvd_rentals.rental and dvd_rentals.inventory tables:

#### Category Insights: 

1. Perform an anti join to check which column values exist in dvd_rentals.rental but not in dvd_rentals.inventory

```SQL
-- how many foreign keys only exist in the left table and not in the right?
SELECT
  COUNT(DISTINCT rental.inventory_id)
FROM dvd_rentals.rental
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.inventory
  WHERE rental.inventory_id = inventory.inventory_id
); 
```

```SQL 
-- how many foreign keys only exist in the right table and not in the left?
-- note the table reference changes
SELECT
  COUNT(DISTINCT inventory.inventory_id)
FROM dvd_rentals.inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```
There seems to be a single value which is not showing up - let’s investigate which film it is:

```SQL 
SELECT *
FROM dvd_rentals.inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```
Conclusion: although there is a single inventory_id record which is missing from the dvd_rentals.rental table - there might be no issues with this discrepancy as it seems that some inventory might just never be rented out to customers at the retail rental stores.

Finally - let’s confirm that both left and inner joins do not differ at all when we look at the resulting row counts from the joint tables:

```SQL 
DROP TABLE IF EXISTS left_rental_join;
CREATE TEMP TABLE left_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM dvd_rentals.rental
LEFT JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id;

DROP TABLE IF EXISTS inner_rental_join;
CREATE TEMP TABLE inner_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id;

-- Output SQL
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)
UNION
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM inner_rental_join
);
```
We also need to investigate the relationships between the actor_id and film_id columns within the dvd_rentals.film_actor table.

Intuitively - we can hypothesise that one single actor might show up in multiple films and one film can have multiple actors. This is known as a many-to-many relationship.

Let’s perform some analysis on the data tables to see if our hunch is on point:

```SQL 
WITH actor_film_counts AS (
  SELECT
    actor_id,
    COUNT(DISTINCT film_id) AS film_count
  FROM dvd_rentals.film_actor
  GROUP BY actor_id
)
SELECT
  film_count,
  COUNT(*) AS total_actors
FROM actor_film_counts
GROUP BY film_count
ORDER BY film_count DESC;
```
Let’s also confirm that there are multiple actors per film also (although this should be pretty obvious!):

```SQL 
WITH film_actor_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT actor_id) AS actor_count
  FROM dvd_rentals.film_actor
  GROUP BY film_id
)
SELECT
  actor_count,
  COUNT(*) AS total_films
FROM film_actor_counts
GROUP BY actor_count
ORDER BY actor_count DESC;
```
#### A - Analysis
This is where we start showing off our arsenal of data analysis techniques in a very structured and systematic manner.

1. Create a base dataset and join all relevant tables

```SQL 
DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT 
rental.customer_id, 
inventory.film_id,
film.title, 
category.name as category_name,
 -- also included rental_date for sorting purposes
rental.rental_date
FROM dvd_rentals.rental
INNER JOIN  dvd_rentals.inventory 
ON rental.inventory_id= inventory.inventory_id
INNER JOIN dvd_rentals.film
ON inventory.film_id=film.film_id
INNER JOIN dvd_rentals.film_category
ON film.film_id=film_category.film_id
INNER JOIN dvd_rentals.category
ON film_category.category_id= category.category_id; 
```

2. Calculate customer rental counts for each category
```SQL 
DROP TABLE IF EXISTS category_counts;
CREATE TEMP TABLE category_counts AS
SELECT
customer_id,
category_name,
COUNT(*) AS rental_count,
MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
GROUP BY
customer_id,
category_name;
```
3. Aggregate all customer total films watched

```SQL 
DROP TABLE IF EXISTS total_counts;
CREATE TEMP TABLE total_counts AS
SELECT
customer_id,
SUM(rental_count)
FROM category_counts
GROUP BY customer_id; 
```

4. Identify the top 2 categories for each customer

```SQL 
DROP TABLE IF EXISTS top_categories;
CREATE TEMP TABLE top_categories AS
WITH ranked_cte AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY
        rental_count DESC,
        latest_rental_date DESC,
        category_name
    ) AS category_rank
  FROM category_counts
)
SELECT * FROM ranked_cte
WHERE category_rank <= 2;
```

5. Calculate each category’s aggregated average rental count

```SQL
DROP TABLE IF EXISTS average_category_count;
CREATE TEMP TABLE average_category_count AS 
SELECT category_name, FLOOR(AVG(rental_count)) AS category_average
FROM category_counts
GROUP BY category_name;  
```
6. Calculate the percentile metric for each customer’s top category film count

```SQL 
DROP TABLE IF EXISTS top_category_percentile;
CREATE TEMP TABLE top_category_percentile AS
WITH calculated_cte AS (
SELECT
  top_categories.customer_id,
  top_categories.category_name AS top_category_name,
  top_categories.rental_count,
  category_counts.category_name,
  top_categories.category_rank,
  PERCENT_RANK() OVER (
    PARTITION BY category_counts.category_name
    ORDER BY category_counts.rental_count DESC
  ) AS raw_percentile_value
FROM category_counts
LEFT JOIN top_categories
  ON category_counts.customer_id = top_categories.customer_id
)
SELECT
  customer_id,
  category_name,
  rental_count,
  category_rank,
  CASE
    WHEN ROUND(100 * raw_percentile_value) = 0 THEN 1
    ELSE ROUND(100 * raw_percentile_value)
  END AS percentile
FROM calculated_cte
WHERE
  category_rank = 1
  AND top_category_name = category_name;
```

7. Generate our first top category insights table using all previously generated tables

```SQL
 DROP TABLE IF EXISTS first_category_insights;
CREATE TEMP TABLE first_category_insights AS
SELECT
  base.customer_id,
  base.category_name,
  base.rental_count,
  base.rental_count - average.category_average AS average_comparison,
  base.percentile
FROM top_category_percentile AS base
LEFT JOIN average_category_count AS average
  ON base.category_name = average.category_name;
```

8. Generate the 2nd category insights

```SQL 
DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS
SELECT
  top_categories.customer_id,
  top_categories.category_name,
  top_categories.rental_count,
  -- need to cast as NUMERIC to avoid INTEGER floor division!
  ROUND(
    100 * top_categories.rental_count::NUMERIC / total_counts.total_count
  ) AS total_percentage
FROM top_categories
LEFT JOIN total_counts
  ON top_categories.customer_id = total_counts.customer_id
WHERE category_rank = 2;
```

#### Category Recommendations:

1. Film counts: 

```SQL 
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS
SELECT DISTINCT
  film_id,
  title,
  category_name,
  COUNT(*) OVER (
    PARTITION BY film_id
  ) AS rental_count
FROM complete_joint_dataset;
```

2. Category Film Exclusions

```SQL 
DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS
SELECT DISTINCT
  customer_id,
  film_id
FROM complete_joint_dataset;
```
3. Final Category Recommendations

```SQL 
DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS
WITH ranked_films_cte AS (
  SELECT
    top_categories.customer_id,
    top_categories.category_name,
    top_categories.category_rank,
    -- why do we keep this `film_id` column you might ask?
    -- you will find out later on during the actor level recommendations!
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() OVER (
      PARTITION BY
        top_categories.customer_id,
        top_categories.category_rank
      ORDER BY
        film_counts.rental_count DESC,
        film_counts.title
    ) AS reco_rank
  FROM top_categories
  INNER JOIN film_counts
    ON top_categories.category_name = film_counts.category_name
  -- This is a tricky anti-join where we need to "join" on 2 different tables!
  WHERE NOT EXISTS (
    SELECT 1
    FROM category_film_exclusions
    WHERE
      category_film_exclusions.customer_id = top_categories.customer_id AND
      category_film_exclusions.film_id = film_counts.film_id
  )
)
SELECT * FROM ranked_films_cte
WHERE reco_rank <= 3;
```

#### R - Report
Once we’ve finished our analysis we want to package up our various code snippets into a single process and generate the final outputs we need to solve our business problem.