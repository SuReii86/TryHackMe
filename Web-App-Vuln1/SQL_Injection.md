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

### FLAG IN TASK 2 *(hint)*:
It just about the definition and the name of the query operators or specific (**built-in**) database.

## Task 3: What is SQL Injection

This task will dive deep into the definition what is SQL, how web apps using it, where it's vulnerable lives in the system and kinds of SQL Injection

One More Time, SQL Injection occur when the input pass straight to the query without proper sanitization or parameterisation. Treating the user-input as the true sql-code query allow attacker to alter the query's logic to interact with the database in ways that developers never intended.

### How Web Applications Use SQL

When you browse a website, there are many page that generated dynamically (automatically update when some value change on your user-related data). 

The example above was about the login form, which can help you bypass the authenticate process.[login](#comments)

This part showing sample about quering to other ID of blog articles

When you visit *http://website.thm/article?id=1*

The database using the query below to return the article has id = *1* from URL 

```sql
SELECT * FROM articles WHERE id = 1 AND public = 1;
```

Look into the code, there are no effective blocking and Authorization just checking if the context is public or not.. This mean you can read every article on the databases

### Where the Vulnerability Lives

**How to notice that flaw?** 

the problem arises when the application build the query to directly concatcenating the user input into the SQL string, If the server-side code look like this:

```sql
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```

If you using the [comment](#comments) technique, there will no more secure to query 

```sql
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
```

the *OR 1=1* always return *TRUE* to the WHERE operator and the comment *--* skip the public check. So it will reveal every article even the private ones.


### Three Types of SQL Injection

- In-Band SQL Injection: the most straightforward type, easiest to notice since it response even detail errors or attach the sensitive query to the original to retrieve more data. So it divide into two sub-types:

    - Error-Based: Depend on which errors it return, searching for CVE or PoC of injection where it's structure information returned.
    - Union-Based: This is the [UNION](#union) technique, I describe above.

- Blind SQL Injection: This type is most relevant when we testing in reality, The website doesn't return any detail output about it's structure. So we guessing and use built-in function to create our own flag to notice if that working or not. (Also luck is really necessary in this type, you could get it in the first time or never get that @@). Also there are 3 common indirect signal:

    - Authentication Bypass: You success to login or don't get an invalid response after submit the malicious input

    - Boolean-Based: Sometimes you just need a minium impact while enumerate the database so using conditional flag is better choice. Response change subtly (content, True/false,...) whether a condition is true.

    - Time-based: Sleep() is another built-in function, it delay the response it that working or having value to return. so depend on how long the response is, you can determine which input is value.


- Out-of-Band Injection: The Last Option to check whether the web app having SQL injection or not. It causes the databases server to make external request that exfiltrates data through seperate channel.

### Detecting SQL Injection

Test every input that interact with the database. Common points include URL parameter, form fields, cookies or HTTP headers.

The simpliest detection method is inject test characters and observe the response

- Enter a single quote ':  if the application returns a database error, the input is likely being inserted into a SQL query without proper handling.
- Try " (double quote): some queries use double quotes instead of single quotes.
- Enter ;--:  if the application behaves differently (e.g., returns different content), the comment syntax is being processed.
- Test OR 1=1: if it changes the results, the input is directly in the query's logic.

Not every test will produce a visible error. If the application suppresses errors, you may need to rely on behavioural differences (Boolean-Based) or timing delays (Time-Based) to confirm injection. We will cover each of these techniques in detail over the following tasks.

### FLAG IN TASK 3 *(hint)*:
Its about the characters to detect SQL Injection vulnerability and which SQL Injection result in the asked output.

## Task 4: In-Band SQL Injection

As I described, This type is the simpliest injection to detect and manipulate output. Because It's "In-Band" which is in the same communication where the normal response come through. Watching the exfiltrated data on it own page response.

Let's discover two sub-types, we have talk about recently: Error-Based and UNION-Base.

### Error-Based

As it's name, we collect data from Error, we not collect data from storage, we collect about it's structure, how it build, which version is it and also how the query format required to complete the request.

We mentioned above the essential characters to SQL Injection is let the single-quote *'* being send in the query to server. What If we  putting it in the input but it still result in error.

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' at line 1 
```

**It Not Useless**

It telling you are interacting with MySQL server and your input are wrapped in another single quotes. So you can craft more precise payloads to extract information by deliberate error messages.

### UNION-Based

Back to the purpose of enumerate data, pulling data from any tables that exists on server that user have access to it. There is the step-by-step guide:

1. **Determine the number of columns**: The *UNION* require both queries have the same number of columns. So to discover how much we need let use this one: (Keep increase the column one by one until the error disappear or response change).

```sql
1 UNION SELECT 1          -- error (wrong column count)
1 UNION SELECT 1,2        -- error (still wrong)
1 UNION SELECT 1,2,3      -- success! The table has 3 columns
```

2. **Identify which column are displayed**

Not all columns may be rendered on the page. Change the original query's value to something that returns no results (like 0), so only the UNION output is displayed:

```sql
0 UNION SELECT 1,2,3
```

**We going stage by stage, from the biggest to detail information**

Database Name --> Table Name --> Table Data

3. **Extract the database name**

In this case, we know columns 3 is visible to us
let change the value 3 to *database()* so we can extract database name.

```sql
0 UNION SELECT 1,2,database()
```

4. **Enumerate tables**

After knowing the database name, we using the built-in table to retrive more information, this time *information_schema.tables()*

```sql
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables() WHERE table_schema = 'database_name'
```

5. **Enumerate columns**

Next, we discover columns on tables that we extracted.

```sql
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE column_schema = 'table_name'
```

Each step from 3 to 5, related to each other it, so we need to calm, stay on track to exfil the required data. Why we guessing when we have a way to get the precise information.

6. **Extract Data**
remember using group_concat() to collect data in single line.
```sql
0 UNION SELECT 1,2, group_concat(username,':',password SEPERATOR '<br>') FROM target_table
```

Understanding why each step works is more important than memorising the payloads. The column count must match because that is how SQL's UNION operator is defined. We use 0 or -1 as the ID because we need the original query to return an empty result, so our injected results are what the application renders. We use information_schema because it is the database's own catalogue of its structure.

### FLAG IN TASK 4 *(hint)*:

Questions asking you about how you detect which type of In-Band Injection it is and How you can manipulate some global value on database that running on that server.

## Task 5: Blind SQL Injection: Authentication Bypass

Why is it calling "Blind" - Because the response don't give you anything hint or specific information about what that web app are running on server. You have to guessing a lot while facing with this type (Mostly in real life).

The most intuitive example about this type is Authentication Bypass because it will working if you can log in the website and never see what output the database response to you (It's never because you just submit the login form with technique to skipping the password requirement not query for data or something else).

#### How Authentication Queries Work

The simple query structure for login form can be like this:
```sql
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```

The application just check whether this query results return any rows. If it does, you log in, if not you stay on the log in page or getting redirect to guest side, with simple announce: "Invalid credentials"

#### How to attack this query
Using the [Comment](#comments) technique. Remember this technique use to cutting the query where the input is, it make the query skipping all the requirement behind that format *--*.

But If your input *(In this case: username)* not available too, you need to make it 100% true by adding *OR 1=1* before the comment.



However, you know the existed username *("admin")*, you just put that in and confirm you are in that account.

And also don't forget the single-quote.

#### Variations

The exact payload depends on the query structure. Some things to try:

- *' OR 1=1;--* is classic bypass, works when the username is wrapped in single quotes

- *' OR 1=1#* this uses # as the comment character (MySQL alternative)

- *" OR 1=1--* for queries that use double quotes around the input

Try both the username and password fields:  some applications only concatenate one of them into the query, so the vulnerable field may vary

#### FLAG IN TASK 5 *(hint)*:
Just the boolean condition you must use if you don't the existed input you need.