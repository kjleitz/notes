# SQL notes

Assume notes pertain to sqlite3 unless otherwise specified.

---
**Table of contents:**

[toc]

---

## sqlite3

Enter the sqlite3 console by doing:

```
user@compy:~$ sqlite3
```

or

```
user@compy:~$ sqlite3 some_database.db
```

Make sure to exit with the `.quit` command.


## Syntax

- statements are terminated by a semicolon
- single quotes are escaped by doubling them up `''`

### Conventions

- use lower case and snake_case for column names
- use single quotes for strings, double rarely for object/column names
- name your join table `table1_table2` (e.g. `cats_owners`, `students_teachers`, etc.)

## Datatypes

sqlite3 has four basic datatypes:

```sql
TEXT    -- basically just string
INTEGER -- integer, obv
REAL    -- decimal (max 15 chars long; 'double precision')
BLOB    -- holds binary data
```

But they hold more datatypes. It's weird. More info can be found [here](http://www.sqlite.org/datatype3.html).

### Booleans

There is a `BOOLEAN` column type in sqlite3, but the values are 0 or 1 (as literal integers), so there is no real `BOOLEAN` datatype, per se. You can declare things `BOOLEAN`, though, and as far as I can tell it's good practice to do so (as opposed to declaring them `INTEGER`).

## Constructing tables

### Creating a table

Create a table like this:

```sql
CREATE TABLE cats(
  id INTEGER PRIMARY KEY,
  name TEXT,
  age INTEGER
);
```

### Adding columns to a table

Add columns to a table like this:

```sql
ALTER TABLE cats ADD COLUMN breed TEXT;
```

### Changing the name of a column

That's hard to do. Best to delete and re-create the table.

## Viewing tables

Within the sqlite3 console, you can use `.tables` to see your tables:

```
sqlite> .tables
cats        test_table
```

and you can see schema of your tables with `.schema` (optional table specified):

```
sqlite> .schema
CREATE TABLE cats(
  id INTEGER PRIMARY KEY,
  name TEXT,
  age INTEGER,
  breed TEXT
);
```

## Deleting tables

Delete a table like this:

```sql
DROP TABLE cats;
DROP TABLE IF EXISTS cats;
```

## Manipulating data

### The `INSERT INTO` command

Insert data like so:

```sql
INSERT INTO cats (name, age, breed) VALUES ('Maru', 3, 'Scottish Fold');
```

The `id`, since it is a primary key, auto-increments.

If there is no value for a certain column, you can use the `NULL` keyword:

```sql
INSERT INTO cats (name, age, breed) VALUES (NULL, NULL, "Tabby");
```

### The `SELECT...FROM` command

Select from a table like this:

```sql
SELECT /*column names*/ FROM /*table to select from*/;
-- like this:
SELECT id, name, age, breed FROM cats;
SELECT * FROM cats;
SELECT name FROM cats;
```

You can also specify column names from specific tables. This is important if you need to distinguish between ambiguous column names when selecting from different tables:

```sql
SELECT name FROM cats, dogs;
-- throws an error:
Error: ambiguous column name: name

SELECT cats.name, dogs.name FROM cats, dogs;
-- prints:
name        name
----------  ----------
Maru        Clifford
Hana        Clifford
-- which isn't the most useful example, but still
```

#### The `AS` keyword (aliasing)

You can also alias column names with `AS` like so:

```sql
SELECT name AS 'Cat Name' FROM cats;
-- prints:
Cat Name
----------
Kirk
Maru
Hana
Li'l Bub
Moe
Patches
```

#### The `DISTINCT` keyword

Select unique values with `SELECT DISTINCT...FROM`:

```sql
SELECT DISTINCT breed FROM cats;
```

It seems to apply to all listed column names, returning duplicates of column names _only if_ at least one column is distinct from the other entries. So like:

```sql
SELECT DISTINCT name, breed FROM cats;
```

...will return two rows with the same `name` as long as they are of different `breed`s, and vice versa.

#### The `WHERE` clause

The `WHERE` clause allows you to give a conditional for what you want to select:

```sql
SELECT * FROM /*table name*/ WHERE /*column name*/ = /*value*/;
SELECT * FROM cats WHERE age = 3;
SELECT * FROM cats WHERE age < 2;
SELECT * FROM cats WHERE name = 'Maru';
SELECT * FROM cats WHERE breed IS NULL;
```

<a name="having"></a>
#### The `HAVING` clause

If you want to use an aggregate function in your condition, use the `HAVING` keyword instead of `WHERE`, like this:

```sql
SELECT breed, COUNT(breed)
  FROM cats
  GROUP BY breed
  HAVING COUNT(breed) > 1;
```

#### The `BETWEEN...AND` modifier

Select between multiple values like so:

```sql
SELECT * FROM cats WHERE age BETWEEN 3 AND 10;
```

#### The `ORDER BY` modifier

You can order `SELECT` output like this (default is ascending order):

```sql
SELECT * FROM [table name] ORDER BY [column name] ASC|DESC;
SELECT * FROM cats ORDER BY name;
SELECT * FROM cats ORDER BY age DESC;
```

#### The `LIMIT` modifier

You can select the "extreme" of a `SELECT` (like, the oldest, the first alphabetically, the youngest three, etc.) like so:

```sql
SELECT * FROM cats ORDER BY age DESC LIMIT 1;
SELECT * FROM cats ORDER BY name LIMIT 1;
SELECT * FROM cats ORDER BY age LIMIT 3;

-- The following two statements are equal:
SELECT * FROM cats ORDER BY age LIMIT 2, 4;
SELECT * FROM cats ORDER BY age LIMIT 4 OFFSET 2;
-- they list four rows after skipping the first two.
```

### The `UPDATE...SET...WHERE` command

Change an entry in the table with an `UPDATE` statement:

```sql
UPDATE [table name] SET [column name] = [new value] WHERE [column name] = [value];
UPDATE cats SET name = 'Marucat' WHERE name = 'Maru';
```

### The `DELETE FROM...WHERE` command

Delete an entry in the table with a `DELETE` statement:

```sql
DELETE FROM [table name] WHERE [column name] = [value];
DELETE FROM cats WHERE id = 3;
DELETE FROM cats WHERE name = 'Maru';
```

## Relating tables

### Foreign keys

Use a foreign key column to relate that entry to an entry outside its table (just make the column `foreign_key` or whatever and of type `INTEGER`, so it can be some other table entry's primary key).

### The `INNER JOIN` query

An `INNER JOIN` returns the rows from both tables you are querying, where a certain condition is met. It's done like this:

```sql
SELECT column_name(s)
FROM first_table INNER JOIN second_table
ON first_table.column_name=second_table.column_name;
```

For example:

```sql
SELECT cats.name, owners.name
  FROM cats INNER JOIN owners
  ON cats.owner_id = owners.id;
  
-- prints:
name        breed       name
----------  ----------  ----------
Kirk        Tabby       Sophie
Maru        Scottish F  mugumogu
Hana        Tabby       mugumogu
Patches     Calico      Sophie
Grumpy Cat  Persian     Sophie
```

### The `LEFT JOIN` query

A `LEFT JOIN` or `LEFT OUTER JOIN` returns _all_ the rows of the left table regardless of whether the condition is met, and it will also return _matched_ data from the right table (unfilled columns will contain `NULL`):

```sql
SELECT cats.name, owners.name
  FROM cats LEFT JOIN owners
  ON cats.owner_id = owners.id;
  
-- prints:
name        breed       name
----------  ----------  ----------
Kirk        Tabby       Sophie
Maru        Scottish F  mugumogu
Hana        Tabby       mugumogu
Li'l Bub    American S
Moe         Tabby
Patches     Calico      Sophie
            Tabby
Grumpy Cat  Persian     Sophie
```

### The `RIGHT JOIN` query

Just the logical reverse of the `LEFT JOIN`, but not supported in sqlite3. Might be in other engines, though (Postgres, etc.).

### The `FULL OUTER JOIN` query

Combines the results of `LEFT JOIN` and `RIGHT JOIN` (basically just gives back all the data from both tables, combined with the `ON` where possible). I don't think this is in sqlite3 either.

### Making a "join table"

Have a many-to-many relationship? Don't wanna keep adding columns? You don't have to! With the all-new join table, your troubles are no more!

```sql
CREATE TABLE cats_owners (
  cat_id INTEGER,
  owner_id INTEGER
);

INSERT INTO cats_owners VALUES (3, 2);
INSERT INTO cats_owners VALUES (3, 3);
INSERT INTO cats_owners VALUES (1, 2);
-- now cat #3 belongs to owners #2 and #3,
-- owner #2 owns cat #1 and #3,
-- owner #3 also owns cat #3, etc.
```

Grab the names of all the cats owner #2 has with the following:

```sql
SELECT cats.name
  FROM cats INNER JOIN cats_owners
  ON cats.id = cats_owners.owner_id
  WHERE cats_owners.owner_id = 2;
```

Or, more generally:

```sql
SELECT column_name
  FROM table_one INNER JOIN join_table
  ON table_one.column_name = join_table.column_name
  WHERE join_table.column_name = condition;
```

## Aggregate Functions

The SQL aggregate functions, as their title suggests, are used to retrieve minimum and maximum values from a column, to sum values in a column, to get the average of a column values, or to simply count a number of records according to a search condition (or lack thereof).

- Taken from [here](http://www.sqlclauses.com/sql+aggregate+functions). Also see [here](http://zetcode.com/db/sqlite/select/) for more info.

### The `COUNT()` function

Count matching records like so:

```sql
SELECT COUNT(*) FROM cats WHERE age BETWEEN 3 AND 10;
SELECT COUNT(*) FROM cats WHERE net_worth > 1000000;

SELECT COUNT(*) FROM cats WHERE breed = 'Tabby';
SELECT COUNT(name) FROM cats WHERE breed = 'Tabby';
-- these two seem similar, but the former counts all
-- records where at least one column has a value, and
-- the latter counts only records with a value for name.
```

#### The `GROUP BY` modifier

To run aggregate functions on grouped sets of data within a table, use `GROUP BY` like so:

```sql
SELECT breed, COUNT(*) FROM cats GROUP BY breed;
-- prints:
breed               COUNT(breed)
------------------  ------------
American Shorthair  1
Calico              1
Scottish Fold       1
Tabby               4

SELECT breed, owner_id, COUNT(breed) FROM cats GROUP BY breed, owner_id;
-- prints:
breed               owner_id    COUNT(*)                                                                                                         
------------------  ----------  ----------                                                                                                       
American Shorthair              1                                                                                                                
Calico                          1                                                                                                                
Scottish Fold       1           1                                                                                                                
Tabby                           3                                                                                                                
Tabby               1           1
```

...if that makes sense.

### The `AVG()` function

Get the average of a column like so:

```sql
SELECT AVG(column_name) FROM table_name;
SELECT AVG(net_worth) FROM cats;
SELECT AVG(net_worth) FROM cats GROUP BY breed;
```

### The `SUM()` function

Get the sum of a column like so:

```sql
SELECT SUM(column_name) FROM table_name;
SELECT SUM(net_worth) FROM cats;
SELECT SUM(net_worth) FROM cats GROUP BY breed;
```

### The `MIN()` and `MAX()` functions

Get the minimum or maximum values of a column like so:

```sql
SELECT MIN(net_worth) FROM cats;
SELECT MAX(net_worth) FROM cats;
SELECT name, MAX(age) FROM cats;
```

### The `LENGTH()` function

Get the length of a text field like so:

```sql
SELECT name, LENGTH(name) FROM cats;
```

### Using aggregate functions in conditionals

If you want to include an aggregrate function in your `WHERE` clause, you need to use `HAVING` instead. See [this section](#having).

## Writing and executing SQL files against a database

Write your SQL queries/statements in a file with the extension `.sql`, then execute them against a database file (eith extension `.db`) like so:

```
user@compy:~$ sqlite3 some_database.db < add_stuff.sql
user@compy:~$ sqlite3 some_database.db < some_queries.sql
```

## Useful commands at the sqlite3 prompt

```
sqlite> .tables
# lists the tables in the database

sqlite> .schema
# shows table schema (columns, datatypes, etc.)

sqlite> .header on
# show the column names when you SELECT

sqlite> .mode column
# allows you to customize column width with the next couple commands

# After the last two commands, a SELECT looks like this:
id          name        age         breed       owner_id
----------  ----------  ----------  ----------  ----------
1           kirk        13          tabby
2           Maru        3           Scottish F  1
3           Hana        1           Tabby       1
4           Li'l Bub    5           American S
5           Moe         10          Tabby
6           Patches     2           Calico

sqlite> .width auto
# adjusts and normalizes column width when displaying

sqlite> .width NUM1, NUM2
# sets custom widths instead
```