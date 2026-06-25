SQL Injection — Querying the Database Type and Version on MySQL and Microsoft

Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft
Series: SQL Injection — Lab 4
Level: Practitioner
Database: MySQL
Goal: Make the application display the database version string.

Recon

Browse the shop. Clicking a product category sends:

GET /filter?category=Gifts


The category parameter goes into the database query. That is the injection point.

Send the request to Burp Repeater so we can edit and resend it.

Response:

HTTP/2 500 Internal Server Error

A single quote should never crash a site. This 500 means our input lands inside the SQL query, raw. Injection confirmed.

<img width="2165" height="899" alt="image" src="https://github.com/user-attachments/assets/6eefdd44-adfa-4e6a-9c85-7c6dc543a61e" />


Step 2 — Count the columns with ORDER BY

UNION needs the same number of columns on both sides. Count them first.

ORDER BY n sorts by column number n. Name a column that does not exist and the database errors out.

<img width="2168" height="858" alt="image" src="https://github.com/user-attachments/assets/f405edc9-0141-48c7-ab7d-6158aa3f029e" />

Result:


ORDER BY 2 → 200 OK
ORDER BY 3 → 500 error

There is no third column. The query returns 2 columns.

Note on the comment: -- - is a SQL comment. It tells the database to ignore everything after it, so the leftover part of the original query cannot break our line. MySQL needs a space after --, which is why it is dash-dash-space-dash.

Step 3 — Confirm the UNION fits

Send a UNION with two NULL columns. NULL fits any column type, so nothing clashes.

Dialect note: Oracle refuses a SELECT with no table and needs FROM dual. MySQL does not. A bare SELECT works.

GET /filter?category=Gifts' UNION SELECT NULL,NULL-- -

Page loads clean. The UNION fits.

<img width="2159" height="846" alt="image" src="https://github.com/user-attachments/assets/c8069acc-eb98-4f64-aa8d-c41cb08f6640" />


Step 4 — Read the version

MySQL stores its version in @@version. Put it in the first column.

GET /filter?category=Gifts' UNION SELECT @@version,NULL-- -

What fell out

The page printed the version string where a category name should be:

8.0.42-0ubuntu0.20.04.1

This means: MySQL 8.0.42, running on Ubuntu 20.04. Lab solved.

<img width="2154" height="883" alt="image" src="https://github.com/user-attachments/assets/aefef0de-986f-48b4-8fc0-b3571eb8ef08" />


Why this works

The app builds its SQL by pasting input straight into the query:


SELECT name, price FROM products WHERE category = 'INPUT_HERE'

When the input becomes Gifts' UNION SELECT @@version,NULL-- -, the database reads it as a new command, not a category name. The fix is a parameterized query, where input is locked as data and can never run as code.
