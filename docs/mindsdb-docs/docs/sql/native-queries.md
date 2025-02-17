# How to Run Native Queries in MindsDB

The underlying database engine of MindsDB is MySQL. However, you can run queries native to your database engine within MindsDB.

## Connect your Database to MindsDB

To run queries native to your database, you must first connect your database to MindsDB using the `CREATE DATABASE` statement.

```sql
CREATE DATABASE example_db
WITH ENGINE = "postgres",
PARAMETERS = {
    "user": "demo_user",
    "password": "demo_password",
    "host": "3.220.66.106",
    "port": "5432",
    "database": "demo"
};
```

Here we connect the `example_db` database, which is a PostgreSQL database.

## Run Queries Native to your Database

Once we have our PostgreSQL database connected, we can run PostgreSQL-native queries.

### Querying

To run PostgreSQL-native code, we must nest it within the `SELECT` statement like this:

```sql
SELECT * FROM example_db (
    SELECT 
        model, 
        year, 
        price, 
        transmission, 
        mileage, 
        fueltype, 
        mpg, -- miles per galon
        ROUND(CAST((mpg / 2.3521458) AS numeric), 1) AS kml, -- kilometers per liter
        (date_part('year', CURRENT_DATE)-year) AS years_old, -- age of a car
        COUNT(*) OVER (PARTITION BY model, year) AS units_to_sell, -- how many units of a certain model are sold in a year
        ROUND((CAST(tax AS decimal) / price), 3) AS tax_div_price -- value of tax divided by price of a car
    FROM demo_data.used_car_price
);
```

On execution, we get:

```sql
+-----+----+-----+------------+-------+--------+----+----+---------+-------------+-------------+
|model|year|price|transmission|mileage|fueltype|mpg |kml |years_old|units_to_sell|tax_div_price|
+-----+----+-----+------------+-------+--------+----+----+---------+-------------+-------------+
| A1  |2010|9990 |Automatic   |38000  |Petrol  |53.3|22.7|12       |1            |0.013        |
| A1  |2011|6995 |Manual      |65000  |Petrol  |53.3|22.7|11       |5            |0.018        |
| A1  |2011|6295 |Manual      |107000 |Petrol  |53.3|22.7|11       |5            |0.02         |
| A1  |2011|4250 |Manual      |116000 |Diesel  |70.6|30  |11       |5            |0.005        |
| A1  |2011|6475 |Manual      |45000  |Diesel  |70.6|30  |11       |5            |0            |
+-----+----+-----+------------+-------+--------+----+----+---------+-------------+-------------+
```

The first line (`SELECT * FROM example_db`) informs MindsDB that we select from a PostgreSQL database. After that, we nest a PostgreSQL code within brackets.

### Creating Views

We can create a view based on a native query.

```sql
CREATE VIEW cars (
    SELECT * FROM example_db (
        SELECT 
            model, 
            year, 
            price, 
            transmission, 
            mileage, 
            fueltype, 
            mpg, -- miles per galon
            ROUND(CAST((mpg / 2.3521458) AS numeric), 1) AS kml, -- kilometers per liter
            (date_part('year', CURRENT_DATE)-year) AS years_old, -- age of a car
            COUNT(*) OVER (PARTITION BY model, year) AS units_to_sell, -- how many units of a certain model are sold in a year
            ROUND((CAST(tax AS decimal) / price), 3) AS tax_div_price -- value of tax divided by price of a car
        FROM demo_data.used_car_price
    )
);
```

On execution, we get:

```sql
Query OK, 0 rows affected (x.xxx sec)
```

Here we have two nesting levels: the native query is nested within the `SELECT` statement, and the `SELECT` statement is nested within the `CREATE VIEW` statement.
