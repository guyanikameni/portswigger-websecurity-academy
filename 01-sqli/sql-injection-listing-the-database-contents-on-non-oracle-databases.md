SQL Injection — Listing the Database Contents on Non-Oracle Databases

Lab: SQL injection attack, listing the database contents on non-Oracle databases
Series: SQL Injection — Lab 5
Level: Practitioner
Database: PostgreSQL
Goal: Dump the usernames and passwords from a hidden table and log in as administrator.

What this bug is

The site has a category filter. Click a category, the page reloads with matching products.

Behind that click, the server glues the category value straight into a SQL query. No filtering. So I can attach my own SQL using UNION — a keyword that stacks a second query's rows under the first. The page shows products. Run UNION in a second query, and my data shows up on the same page.

There is a hidden table with usernames and passwords. I just steer the bug toward it.
Step 1 — Break the query

I opened the lab in Burp Suite (sits between browser and site, lets me read and edit every request) and clicked a category:

%2c is the encoded comma in the category name. Sent it to Repeater.

Add one single quote to the category value:

?category=Clothing%2c+shoes+and+accessories'

The page errors out. The quote landed inside the SQL and broke its syntax. Injection confirmed.



<img width="2203" height="843" alt="image" src="https://github.com/user-attachments/assets/c9d1abe5-e8ef-4bc4-ab32-a07eb4d033a3" />

Step 2 — Count the columns

UNION only works if my second query has the same number of columns as the first. Count with ORDER BY (sort by column 1, then 2, then 3).


?category=Clothing%2c+shoes+and+accessories'+ORDER+BY+1--
?category=Clothing%2c+shoes+and+accessories'+ORDER+BY+2--
?category=Clothing%2c+shoes+and+accessories'+ORDER+BY+3--

-- comments out the rest of the original query. ORDER BY 2 works, ORDER BY 3 errors. The query returns 2 columns.



<img width="2168" height="841" alt="image" src="https://github.com/user-attachments/assets/e35390f9-6d60-4c5f-bcad-a40fd7a14814" />


Step 3 — Confirm both columns hold text

I will be dumping strings, so I need text columns. Test with markers:

?category=Clothing%2c+shoes+and+accessories'+UNION+SELECT+'abc','def'--

abc and def show on the page. Both columns take text.

<img width="2158" height="779" alt="image" src="https://github.com/user-attachments/assets/affa0792-a9f8-4085-aeab-a332789398f6" />


Step 4 — List every table

GET /filter?category=Accessories'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--HTTP/2 HTTP/2


information_schema.tables is a built-in list of every table in the database. (Oracle calls it all_tables — different dialect, different lab.)

The response fills with names like pg_event_trigger, pg_shdescription. Every pg_ prefix means PostgreSQL. The table names fingerprinted the database for me — no version query needed.


<img width="2177" height="910" alt="image" src="https://github.com/user-attachments/assets/3306e2c4-a7ad-42dc-a4cd-92dab922107f" />


Step 5 — Cut the noise

200 tables is too many. Make the database filter itself:

...+FROM+information_schema.tables+WHERE+table_name+LIKE+'%25users%25'--

LIKE '%users%' = any name with "users" in it. %25 is the encoded %.

One hit: users_zdxxgl. The name is randomized, so you cannot copy a guess — you find it.


<img width="2099" height="872" alt="image" src="https://github.com/user-attachments/assets/061ba747-7a96-408e-886a-68903e214656" />



Step 6 — Dump the credentials

login with the credentials  you found for there admin 


<img width="1086" height="592" alt="image" src="https://github.com/user-attachments/assets/a703793f-eacf-4abd-bd3b-f88245a76d07" />



What fell out

Sent it again. This time the table did not stop at products. At the bottom — usernames against passwords, plain text, right in the response.

Copied the administrator password. Logged in. Lab solved.

[INSERT SCREENSHOT: dumped rows including administrator]


Why this works

information_schema is not a flaw. PostgreSQL, MySQL, and MSSQL all ship it by default.

The flaw is the app trusting the category value and gluing it into the query as text, instead of using a parameterized query (input passed as data, never as code). One quote closes the string early, and the rest of my input runs as SQL.

Devs cause this by reusing a quick query builder for a filter, assuming category names are a fixed list. But the request is fully in the user's hands — dropdown or not.




