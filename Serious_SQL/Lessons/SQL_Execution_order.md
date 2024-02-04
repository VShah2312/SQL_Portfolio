### SQL Logical Execution Order
In a nutshell - all SQL queries are “ran” in the following order:

1. FROM
    -  WHERE filters
    - ON table join conditions
2. GROUP BY
3. SELECT aggregate function calculations
4. HAVING
5. Window functions
6. ORDER BY
7. LIMIT