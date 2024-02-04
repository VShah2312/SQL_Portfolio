# Marketing Analytics Case Study 

## Case study overview: 

Personalized customer emails based off marketing analytics is a winning formula for many digital companies, and this is exactly the initiative that the leadership team at DVD Rental Co has decided to tackle!

We have been asked to support the customer analytics team at DVD Rental Co who have been tasked with generating the necessary data points required to populate specific parts of this first-ever customer email campaign.

Throughout this marketing case study we will cover many SQL data manipulation and analysis techniques. The aim is to further extend your SQL knowledge base and also expose you to some scenarios where you can apply some neat tricks that I’ve picked up over the years!

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


