# SQL - Structured Query Language

The following notes are *A programming language designed to manage data stored in relational databases.* SQL is a *declarative* language (vs. *imperative*). That is, one tells SQL what one wants, not how to get it. 

## Table of Contents

* [Advantages and Disadvantages](#advantages-and-disadvantages)
* [Data Types](#data-types)
* [SQL Statements](#sql-statements)
    * [The `SELECT` Statement](#the-select-statement)
    * [Filter Statements](#filter-statements)
    * [Update Statements](#update-statements)
    * [The `JOIN` Statements](#the-join-statements)
    * [The `CREATE` & `DROP` Statements](#the-create-drop-statements)
* [SQL Functions](#sql-functions)
    * [Aggregate Functions](#aggregate-functions)
    * [String Functions](#string-functions)
    * [Date Functions](#date-functions)
    * [Window Functions](#window-functions)
    * [Other Functions](#other-functions)
    * [`CREATE` Functions](#create-functions)
* [Subqueries](#subqueries)
* [Order of Operations](#order-of-operations)
* [Operators](#operators)
* [The `CASE` Expression](#the-case-expression)
* [Database Adminstration](#database-administration)
    * [Admin Statements](#admin-statements)
    * [Schema](#schema)
* [Temp. Tables](#temp-tables)
* [Vocab](#vocab)


## Advantages and Disadvantages

Structured databases are popular for their advantages, which may also be disadvantages in certain situations.

* **Structured**: tables and schema are fixed and inflexible both an advantage (in terms of user IO) and disadvantage (if a change is needed, the data do not lend themselves to a table format)
* **Centralized**: SQL is designed to be in a single server, rather than distributed across servers. Both an advantage (controlled) and a disadvantage (e.g., moving Asian customer table to Asia for efficiency).

## Data Types

* `CHAR`: fixed-length characters
* `DATE`: date
* `ENUM`: values from a discrete set of options. E.g., `ENUM('android','iphone','other')`
* `INTEGER`: integer.
* `MONEY`: Dollar format
* `TEXT`: string format. Note that this must be inserted into SQL with `'` and escaped with double `'`. E.g., `'String for O''Brian'`.
* `VARCHAR`: variable-length characters

Note that variables may be cast to new formats with the `::` operator. For example, `SELECT '10'::INTEGER;`

## SQL Statements

SQL comes in many 'flavors', but the following statements are generally universal with some exceptions. Some statements (e.g., `COPY`) are more specific to flavor and should be Googled. SQL *statements* are made of *clauses*, written in CAPITALS by convention (not syntactically required), have *parameters* in between `()`, and end in `;`. 

### The `SELECT` Statement

The most common query. `SELECT` is used to select data from a table. For example,
```sql
SELECT var1_name, var2_name FROM table_name;
```

* `*`: `SELECT` all table columns). `SELECT` always returns a *new* table called the *result set*.
* `DISTINCT`: a query returning unique values in a result set. Example: `SELECT DISTINCT genre FROM movies;`
* `AS`: rename column names using a *alias*. Example `SELECT name AS "Album Name", year FROM albums;`. Depending on the flavor of SQL, the alias may need to be in *double* quotes, and `AS` might be unnecessary (e.g., PostgreSQL)
* `UNION`: Combine results from two sub-query result sets, with the same column name, type and ordering. By default, duplicate rows are removed. This may be overridden with a subsequent `ALL` statement. For example: 

```sql 
SELECT * FROM TABLE1
UNION ALL
SELECT * FROM TABLE2
```

Math may be conducted in line for variable in the select statement. For example, `SELECT proportion_sick*100 AS percent_sick FROM table_name;`. 

### Filter Statements

* `GROUP BY`: Clause that aggregates rows of a column name that have the same value. It is only used with other aggregate functions. Usually, the columns you are grouping by are also in the `SELECT` statement. For Example:
```sql
SELECT price, type, COUNT(*) 
FROM fake_apps 
GROUP BY price, type;
```
Note, that order does not matter for multiple `GROUP BY` columns; the statement returns all permutations, regardless. Further note that `GROUP BY 1` is equivalent, that groups by the 1<sup>st</sup> column regardless of what it is called.
* `HAVING`: A `WHERE` clause for aggregate functions. For Example:
```sql
SELECT
    name AS 'Artist',
    SUM(albums) AS 'Number of Albums'
FROM
    artists
GROUP BY 
    name
HAVING
    SUM(albums) > 2;
```
* `ORDER BY`: order the results of a query, either alphabetically or numerically. Example: 
```sql
SELECT * 
FROM movies 
ORDER BY imdb_rating DESC;
```
    * `DESC`: modifier for `ORDER BY` in descending order. See example above. 
    * `ASC`: modifier for `ORDER BY` in ascending order (i.e., `1,2,3,...` or `A,B,C...`).
* `LIMIT`: specifies the maximum number of rows to return. Example: `SELECT * FROM movies ORDER BY imdb_rating DESC LIMIT 3`
* `WHERE`: filters row(s) based on following *conditional* in order to `SELECT`, `SET`, etc. See `UPDATE` example.
    * `AND`: combines `WHERE` conditions. Both conditions must be true to be included in the result set. EXAMPLE: `SELECT * FROM movies WHERE name BETWEEN 1998 AND 2000 AND genre = 'comedy'` 
    * `BETWEEN`: filter results between two values. Example: `SELECT * FROM movies WHERE name BETWEEN 'A' AND 'J'` filters movies with a name starting with 'A' up to, but not including 'J'.
    * `IN`: Specify multiple values for the `WHERE` clause. For example: `WHERE name IN (name_1, name_2, ...)`
    * `LIKE`: Pattern recognition for `WHERE` filtering. Example: `SELECT * FROM movies WHERE name LIKE "Se_en";` returns moves with names `Se7en` *and* `Seven`. `_`, for a single letter, and `%`, for zero or more letters are both wildcards.
    * `OR`: combines `WHERE` conditions similarly to `AND`. However, only one condition must be true to be included in the result set. 
Note that aggregators are not allowed in the `WHERE` statement because `WHERE` occurs before `SELECT` statement and aggregators.

### Update Statements

Update statements change or remove information from SQL databases.

* `ALTER TABLE`: Add new column to a `TABLE` object. Example: `ALTER TABLE table_name ADD COLUMN miley_cirus_fan BIT`.
* `DELETE FROM`: Remove row(s) of data. Example: `DELETE FROM celebs WHERE twitter_handle IS NULL`;
* `INSERT INTO`: Add row to an existing relation. Example: 
```sql
INSERT INTO
    celebs [(id, name, age)] #Specify if non-inserting row values for all columns
VALUES 
    (1, 'Justin Bieber' , 21);
```
* `SET`: indicates a column to edit. See `UPDATE` example.
* `UPDATE`: used to edit a data row in a relation, often in conjunction with `SET` and `WHERE`. Example: 
```sql
UPDATE celebs 
SET age = 22
WHERE id = 1;
```

### The `JOIN` Statements

Tables are related to each other using *primary keys* and *foreign keys*. Primary keys are a unique identifier for rows in a table. Foreign keys are the unique identifier in one table for the rows in a *another* table. 

* `JOIN` & `ON`: Joins two tables into a single table set based `ON` a primary & foreign key pair. This command is called an *inner* command without `LEFT` or `RIGHT`, meaning only rows that exist in *both* tables are returned (i.e., `JOIN` and `INNER JOIN` are equivalent). Example: 
```sql
SELECT 
    albums.id, 
    albums.name AS Album, 
    albums.year, 
    artist.name AS Artist   
FROM
    albums
JOIN artists ON
    albums.artist_id = artists.id AND
    artists.genre != 'pop'; #do not join pop artists
```
* `LEFT` and `RIGHT JOIN`: An outer join command includes `NULL` foreign keys for the left and right table respectively. All rows in the "left" table are returned, regardless of meeting the *join condition*. Example: `...FROM albums LEFT JOIN artists ON albums.artist_id = artists.id;` includes *all* rows from `albums`, regardless of whether `albums.artist_id` contains a value. 
* `FULL JOIN`: An outer join including all rows for each table.

![Join Reference](joins_reference.png)

`JOIN` statements may be chained together for a single, or multiple tables. 

## The `CREATE` & `DROP` Statements

* `DROP`: delete a database or table
* `CREATE`: create a database
    * `CREATE TABLE`: create a database `TABLE`. Example: `CREATE TABLE tab_name(id INTEGER PRIMARY KEY, name TEXT);`
    * `PRIMARY KEY`: tell SQL which column is the unique identifier of a row. SQL provides insurance that the `PRIMARY KEY` is unique for each row and that none of the values are `NULL`.

## SQL Functions

### Aggregate Functions

* `AVG()`: Aggregate function that returns the average value of a column, given as a parameter. Example: `SELECT AVG(price) FROM fake_apps;`.
* `COUNT()`: function that counts the rows of a result set. Example: `SELECT COUNT(*) FROM movies;` returns the number of movies in the database.
* `LENGTH()`: returns the length of text columns. For example: `SELECT name, LENGTH(name) FROM pets;` returns the name and length of name of pets (might be `LEN()` in some SQL implementations)
* `MAX()`: Aggregate function that returns the maximum value for a column, given as a parameter. Example: `SELECT MAX(downloads) FROM fake_apps;`
* `MIN()`: Aggregate function that returns the maximum value for a column, given as a parameter. Example: `SELECT MIN(price) FROM fake_apps;`
* `ROUND()`: Aggregate function that rounds a column to a specified number of digits, both given as parameters. Example: 
```sql
SELECT price, ROUND(AVG(downloads), 2)
FROM fake_apps
GROUP BY price;
```
* `SUM()`: Aggregate function that adds the `INTEGER` or `REAL` values of a column, given as a parameter. Example: `SELECT SUM(downloads) FROM fake_apps;`

### Date Functions

* `DATEDIFF(<date_units>, <date1>, <date2>)`: Calculate the difference in `date_units` between `date1` and `date2`.
* `DATE_PART()`: Filter a specific part of date. E.g., `DATE_PART('year', <date_col>)` extracts the year from `date_col`.
* `TO_DATE()`: converts string literal to date format. For example, `SELECT TO_DATE(<datetime>)`, or `SELECT TO_DATE(<string> , 'MM/DD/YYYY')` where `string` is in the given format. 
* `INTERVAL <value>`: Specifies a time interval. E.g., `duration*interval '1 day'` would convert `duration` integer value into a that number of days.    

### String Functions

* `RIGHT(<str>, <num>)`: Return the right `num` number of characters from `str`. E.g., `RIGHT(account_str, 4)` selects the last `4` characters of `account_str`.
* `CONCAT(<arg1>, <arg2>, ...)`: Concatenate a number or arguments into a single string. 

### Window Functions

Window functions operate on a set of rows and return a single value for each row. The term window refers to the set of rows that the function operates on. There are a number of aggregate functions that are applicable within the window, including `AVG`, `COUNT`, `FIRST_VALUE`, and `SUM`. The syntax is `<FUNCTION()> OVER(PARTITION BY <GROUP_COLUMN>) [AS <alias>] ` the following:

```SQL
SELECT
    merchant_location_id,
    service_date,
    ROW_NUMBER() OVER(PARTITION BY merchant_location_id ORDER BY service_date DESC) AS bill_order
FROM
    raw.analytics.analytics_insights_billing
```

More information on window functions may be found here: [Window Functions](https://docs.aws.amazon.com/redshift/latest/dg/c_Window_functions.html).

### Other Functions

* `CAST(<col_name> AS <type>)`: Cast a column into a new data type.
* `COALESCE()`: exchanges null values for a user-defined value. E.g., `COALESCE(col_name, 0)` turns all null values in `col_name` into `0`. 
* `EXISTS(<query>)`: Returns a boolean value regarding whether the input `<query>` exists.
* `TYPEOF(<col_name)`: Return data type of `col_name` 

### `CREATE` Functions

The user may define their own functions with `CREATE FUNCTION`. For (atleast) PostgreSQL flavor, there are 4 requirements

1. Specify `RETURNS <data_type>`
1. Specify language with `LANGUAGE SQL` (or `LANGUAGE plpgsql`)
1. Use `SELECT` or `BEGIN RETURN ... END` to specify the output
1. `$$` or `'` surrounds function logic

Here is a syntactical example from `codewars.com`:

```SQL
CREATE FUNCTION agecalculator(date) RETURNS integer
AS $$ BEGIN RETURN EXTRACT(YEAR FROM AGE($1))::int END$$
LANGUAGE SQL;
```

Or equivalently

```SQL
CREATE FUNCTION agecalculator(date) RETURNS int
AS 'SELECT EXTRACT(YEAR FROM AGE($1))::int;'
LANGUAGE SQL
```

More information regarding creating custom functions with PostgreSQL's `CREATE FUNCTION` command found [here](https://www.postgresql.org/docs/9.5/static/sql-createfunction.html).

## Subqueries

You can make a subquery to substitute for table in a `FROM` statement. For example, `SELECT name FROM (subquery)`.

## Order of Operations

1. `SELECT [DISTINCT, AGG*] <table1.col1, ..., table1.coln, table2.col1, ...>`
2. `FROM <table1>`
3. `JOIN <table2> ON <table1.key_name> = <table2.other_key_name>`
4. `WHERE <table1.coli = val1> AND <table2.colj> = va2>` 
5. `GROUP BY <table1.colk>`
6. `HAVING <AGG*(table1.col) = val>`
7. `ORDER BY <table1.col> ASC <table2.col> DESC`

## Operators

* `||`: concatenate strings. E.g., `SELECT 'Ron Weasley has ' || COUNT(*) || ' pets'  FROM pets WHERE owner = 'Ron Weasley';`
* `::`: Cast `field` to `fmt`. E.g., `SELECT dollars::FLOAT`.
* `->>`: Access fields in JSON data. E.g., 
```sql
SELECT * FROM email_events
WHERE sendgrid_data->>'event' = 'email_verification_failure'
```
* `--`: comments
* `/* ... */`: Multi-line comments

## The `CASE` Expression

A generic conditional expression that is similar to if/else expressions in other languages. It follows the following basic syntax.

```sql
CASE  
    WHEN condition THEN result
    [ WHEN condition THEN result ]
    ...
    [ ELSE result ]
END
```

Where `condition` is a conditional expression and `result` is the specified return value (braces indicate optional arguments). Alternatively, `CASE` Expressions may follow this syntax:

```sql
CASE expression
    WHEN value THEN result
    [ WHEN value THEN result ]
    ...
    [ ELSE result ]
END
```

where `expression` is an expression that is evaluated and compared with `value` for equality. `CASE` statements may be inserted directly within `SELECT` statements and end with `END AS var_name, ...`.

## Database Administration

### Admin Statements

* `USE <TYPE> <NAME>`: Specify a database or schema for `TYPE` by `NAME` for query
* `SHOW COLUMNS IN <table>`: Used to show `table` columns and information regarding them

### Schema

A meta-SQL table with information regarding tables, databases and more. A common usage of the `information_schema` (postgreSQL) table is to learn about `columns`. For example `information_schema.columns` contains much information on all table's columns, including `column_name` and `data_type`.

```sql
CREATE TABLE table_name(
    id INTEGER, 
    name TEXT,
    age INTEGER
);
```

Where `CREATE TABLE` is a clause and `column_name DATA_TYPE` are parameters. Numbers of lines used does not matter.

### Importing Data With `COPY`

Data may be copied into a SQL database from a local or remote source using the following `COPY ... FROM` syntax:

```sql
COPY <table_name> FROM <path>
DELIMITER ',' CSV
HEADER;
```

## Temp. Tables 

Using `WITH` and `AS` statements, a temporary table may be created *for the current session*. For example:

```sql
WITH calls_made AS 
    (SELECT
        caller_id,
        count(*) AS total_calls
    FROM call_history
    GROUP BY caller_id
    )
SELECT
    caller_id,
    total_calls
FROM 
    calls_made;
```

Temporary tables may also be created with `CREATE TEMPORARY TABLE table_name AS ...` or `SELECT * INTO TEMPORARY TABLE table_name FROM ...`. These temp tables will be dropped upon disconnection.

From the command line, SQL databases may be created with `.sql` text files using the following syntax:

```bash
sqlite3 database.db < create_database.sql
```

A `.db` file may then be queried in a SQL environment.

## Vocab

This should be migrated to another vocabulary list.

* **Child Table**: A table that references another's primary key, using a foreign key. E.g., a `sales_record` table, referencing a `customer` table with a foreign key. If updating a child table ever requires you to update the parent as well, then we have improper design. 
* **Column**: a set of data values of a particular type.
* **Cross-reference Table**: A table that links primary keys from two tables, when there are many-to-many relationships between them. E.g., student applicants to universities tables.
* **Database Normalization**: A database design ideal: *All non-key columns should belong exclusively to the primary key*. 
* **Extract, Transform, Load**: three database functions that are combined into one tool to pull data out of one database and place it into another database.
* **Foreign Key**: A table column that contains *another* table's primary key. A foreign key *can* contain `NULL`.
* **Parent Table**: A table that stores a primary key, referenced by a **child table**'s foreign key. E.g., a cities parent table, and a neighborhoods child table.
* **Primary Key**: A table column that uniquely identifies a row from the table and cannot contain `NULL`. 
* **Row**: a single record in a table
* **Relationship**: A table primary key that is the connection between two or more tables.
* **Relational database**: a database that organizes information into one or more tables. Saves space 
* **Relational Database Management System (RDBMS)**: a program that lets you create, update, and administer a relational database. Most relational database management systems use SQL to access the database. Types: SQLite, MySQL, SQL Database
* **Schema**: defines the structure of a table or a database
* **Structured Data**:
* **Table (AKA Relation)**: a collection of data organized into rows and columns.
