# SQL Injection in WHERE Clause — Retrieval of Hidden Data

**Lab:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
**Series:** SQL Injection — Lab 1
**Level:** Apprentice
**Goal:** Make the shop display products that are normally hidden (unreleased items) by injecting SQL into the category filter.

---

## What this bug is

The shop builds a database question (a SQL query) using the category I click. It pastes my category text straight into the query without cleaning it. So I can stop sending a category name and start sending SQL commands.

The normal query looks like this:

SELECT * FROM products WHERE category = 'Gifts' AND released = 1

The `released = 1` part hides unreleased products. If I can comment that part out or force the WHERE condition to always be true, every product shows — including the hidden ones.

---

## Step 1 — Capture the traffic in Burp

I opened the lab in Burp Suite's built-in browser so all traffic flows through Burp automatically. Then I clicked the **Gifts** category in the shop.

In Burp, I went to **Proxy → HTTP history** and found the request:

<img width="1282" height="1000" alt="image" src="https://github.com/user-attachments/assets/70b7bc7b-61d2-4d3a-9181-d9b7a6b541e6" />

GET /filter?category=Gifts


The `category=Gifts` is a parameter — a value the app reads and uses to look up products. That's my injection point.


---

## Step 2 — Send it to Repeater and get a baseline

I right-clicked the request → **Send to Repeater**, then opened the Repeater tab.

I sent the unchanged request first to record what normal looks like:

<img width="1242" height="811" alt="image" src="https://github.com/user-attachments/assets/78df9e7b-68bc-4b19-89c7-bdb0dcc11f3d" />

Response: `200 OK`. I noted the response length. You can't tell that something changed without knowing normal first.

 Repeater baseline Gifts request, 200 OK


## Step 3 — Probe with a single quote

The single quote `'` is the SQL probe. In SQL, a quote ends a piece of text. If the input is unsafe, my quote breaks the query.

I changed the value to `Gifts'` and sent it:

 GET /filter?category=Gifts' HTTP/2

 <img width="1234" height="758" alt="image" src="https://github.com/user-attachments/assets/dd6f4ba0-5821-45a9-bf64-37821c8c39c4" />

Response:
HTTP/2 500 Internal Server Error
*The database choked. My query had become:

SELECT * FROM products WHERE category = 'Gifts'' AND released = 1
That extra `'` has no partner, so the query is broken syntax. The 500 error only happens because my raw input reached the SQL engine. **Injection confirmed.**


---

## Step 4 — Exploit: force every row true and drop the filter

I changed the value to `Gifts' OR 1=1--` and sent it:

GET /filter?category=Gifts' OR 1=1-- HTTP/2

SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1

- `'` → closes the `Gifts` text.
- `OR 1=1` → always true, so every product row matches.
- `--` → comments out the rest (`' AND released = 1`), killing the released filter.

Response: `200 OK`, with a much larger body — every product in the shop, including unreleased ones.

<img width="1155" height="873" alt="image" src="https://github.com/user-attachments/assets/9de66b29-5d66-4223-91eb-4304c1ec7855" />


---

## Step 5 — Lab solved

The lab banner turned green: **Congratulations, you solved the lab!**


<img width="1083" height="899" alt="image" src="https://github.com/user-attachments/assets/1593c6b1-429c-4b4d-823f-69a670d1b79a" />


---

## Why this works

The app built its SQL query by gluing user input directly into it. When the input is a normal word, the query works. When the input contains SQL syntax like `'` or `OR 1=1`, that syntax becomes part of the command. The database can't tell my data from the developer's code.

**The fix:** parameterized queries (prepared statements). The query structure is defined first, and user input is passed separately as pure data — never as part of the command. Then `Gifts' OR 1=1--` is treated as a literal category name, matches nothing, and the injection fails.

---

## How I'd spot this in a real app

1. Find every input that filters, searches, sorts, or looks up data — URL params, search boxes, login fields, `?id=` values.
2. Don't stop at the URL bar — check POST bodies, JSON, cookies, and logged headers (`User-Agent`, `Referer`, `X-Forwarded-For`).
3. Record a baseline (status code + response length) before testing.
4. Send a single `'`. Watch for any change: a 500 error, a different length, missing or extra results, or a delay.
5. Confirm control with `'--` or `OR 1=1--`, then escalate (UNION) to extract data.
6. Be careful with `OR 1=1` on inputs that might write or delete — it can match every row in an UPDATE/DELETE.

---

**One-line summary:** A single quote broke the query (500 error), and `Gifts' OR 1=1--` forced every row true — dropping the released filter and spilling the shop's hidden products — because the app built its database question out of my raw input.
