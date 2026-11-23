## Task 1: Introduction
*No Flag needed*

SQL Injection is one of the most critical vulnerability, List under OWASP TOP 10 (A05: Injection).

The Condition to achieve SQL Injection is having the input direct to databases query without sanitize or restriction.


The Consequences can be severe: access to sensitive data, modified or deleted records and even the full control of the databases server itself.

### Why SQL Injection still one of the most well-known type of injections in modern structure.

    The Easiest explation is Databases (Data) is one of the most valuable component to attack as a attacker. The Query is not secure even in many modern application with AI to checking query's log.

## Task 2: SQL Essentials for Injection

### Comments

The common syntax to achieve the vulnerality is *comment* : *--*

In Single-Line Query(Maybe there are multi-line but as I have seen the query structure still usng one line to query which occur the flaw):
```sql
SELECT * FROM users WHERE username = 'INPUT' AND password = 'secret';
```

The comment can cut the query to just only use the *username* to query, skipping everything after *--*
as the password never need to authenticate with server.

```sql
SELECT * FROM users WHERE username = 'INPUT'--' AND password = 'secret';
```

#### Why we having the single quotes here, not just the comment syntax
    Although the comment can skip everything after it, but if the query having syntax error. It will never run.

### UNION

Union can let you combine two query to gain read the databases storage, it not the same purpose as the [SQL Comments](#comments), it use to gain data, not bypass authentication.

```sql
SELECT name, age FROM students UNION SELECT username, id FROM admins;
```

Attack can append their own *SELECT* statement to a legitimate query, pulling data from entirely different tables.

This is another type of SQL Injection call: *UNION-Based SQL Injection*.

**Note: This UNION Injection depend on how many columns original query using to select, It also need the same number of values to success.**

### LIKE And Wildcards

As we learn this two components *[LIKE operators with the wildcards (% , _ )]* can use to search for data that we don't remember exactly.

And this injection technique also use to enumerate data as the [UNION-Based](#union)

```sql
SELECT * FROM users WHERE username LIKE 'adm%';
```

In Blind SQL Injection, attackers use LIKE with wildcards to enumerate data one character at a time — testing LIKE 'a%', LIKE 'b%', and so on until they find a match.

### LIMIT

The LIMIT clause limits the number of rows returned. The syntax *LIMIT offset, count* lets you skip rows and control output size.
```sql
SELECT * FROM users LIMIT 1;       -- returns only the first row
SELECT * FROM users LIMIT 2, 1;    -- skips 2 rows, returns the 3rd
```
In injection payloads, LIMIT is often used to control which row is returned or to prevent the output from being overwhelmed by too many results.

### Strings function [CONCAT(), GROUP_CONCAT()]
These function can help you format the output when extracting the data.

- *concat()*: can join individuals value together `CONCAT(username, ':', password)` produces `admin:pass123` for a single row.
- *group_concat()*: aggrerates values in multiple rows into single comma-seperated string, So instead of getting data in multi-row format (sometimes it can occur error to the query result) into single row.
```sql
SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;
-- Returns: admin:pass123<br>martin:secret<br>jim:work456
```

### The information_schema database
This database name is the built-in database in MySQL, MariaDB, PostgreQL. It contain metadata about every other database on the server: database names, table names, column names, data types. 

Two tables within this database is:
- information_schema.tables: list every table.
    - *table_schema* column hold the database name.
    - *table_name column* hold the table name.
- information_schema.columns: list every column. The *table_name* and *column_name* in this table, these columns let you discover the structure of any table (I thinks showing the labels of each column in table)

When performing [Union-Based injection](#union), information_schema is how you go from "I can inject" to "I know every table and column in this database."

### A Note on Database Engines
This room uses MySQL syntax throughout. Other database engines (MSSQL, PostgreSQL, SQLite, Oracle) have their own variations: different comment syntax, different system tables, and different functions. The core concepts transfer, but the exact payloads differ. Once you master MySQL injection, adapting to other engines is straightforward.

### FLAG IN TASK 2:
It just about the definition and the name of the query operators or specific (**built-in**) database.