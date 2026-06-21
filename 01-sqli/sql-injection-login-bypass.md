# SQL Injection — Login Bypass

**Lab:** SQL injection vulnerability allowing login bypass
**Series:** SQL Injection — Lab 2
**Level:** Apprentice
**Goal:** Log in as the administrator without knowing the password, by injecting SQL into the login form.

---

## What this bug is

When you log in, the app checks your details against a database with a query like this:

SELECT * FROM users WHERE username = 'you' AND password = 'yourpassword'


If a row comes back, you're logged in.

The username comes from the login form. The app pastes it into the query without cleaning it. SQL has a comment symbol `--` that means "ignore the rest of this line." If I comment out the password check, the database never checks the password.

---

## Step 1 — Find the login form

I opened the lab in Burp Suite's built-in browser (so all traffic flows through Burp). On the homepage, the **My account** link at the top leads to the login page.

<img width="1081" height="1783" alt="image" src="https://github.com/user-attachments/assets/c3fc652a-d3c7-4aa4-9bb9-864d45b5c5eb" />


---

## Step 2 — Read the form source (find the CSRF token)

Before attacking, I checked the page source in Burp. The form had a hidden field:

<img width="1091" height="902" alt="image" src="https://github.com/user-attachments/assets/fd4230c0-5dd9-4d2f-b6d1-621e7a47cedb" />


input type="hidden" name="csrf" value="9bDGyahIAnJln7oObwEjoiVWM0sLFKDc">

A CSRF token — a one-time value the form sends to prove the request is real. This means the login request has **three** fields: `csrf`, `username`, `password`. I must keep the csrf intact or the server rejects the request before my injection runs.


Step 3 — Capture the real login request
I typed junk credentials (username test, password test) and submitted with Intercept on (Proxy → Intercept). Burp paused the request and showed the body:


<img width="1715" height="895" alt="image" src="https://github.com/user-attachments/assets/a8d392fe-a0e5-4bcc-bb96-3c7d5546aa44" />


Step 4 — Inject into the username
I changed only the username value, leaving csrf and password alone:

csrf=9bDGyahIAnJln7oObwEjoiVWM0sLFKDc&username=administrator'--&password=test

I forwarded it.
What this builds on the backend:

SELECT * FROM users WHERE username = 'administrator'--' AND password = 'test'


<img width="2249" height="799" alt="image" src="https://github.com/user-attachments/assets/cf308627-1dd4-465f-bb95-cb47d6d8f587" />


' → closes the administrator text.
-- → comments out everything after, so AND password = 'test' is ignored.
The database checks only username = 'administrator'. No password check.

Step 5 — The response: bypass confirmed

HTTP/2 302 Found
Location: /my-account?id=administrator
Set-Cookie: session=jxWIZaT8SD4GsJOR1c724DTq2AjoQpUI; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 0

The 302 redirect to /my-account?id=administrator means the login succeeded. The server also handed me a new logged-in session cookie.


Step 6 — The cookie lesson (Repeater vs browser)
In Repeater, following the redirect kept bouncing me back to login. Reason: the server gave a new logged-in cookie in the response, but Repeater kept sending the old one. The cookie is what proves you're logged in — not the redirect.
The fix: do the final login in Burp's browser. With Intercept off, I went to the login page, typed administrator'-- as the username and any password, and clicked Log in. The browser carried the new cookie automatically.


Step 7 — Lab solved
The page loaded as the administrator and the lab banner turned green: Congratulations, you solved the lab!


<img width="2267" height="845" alt="image" src="https://github.com/user-attachments/assets/22e8420a-b812-4419-80a3-c6de53444497" />


Why this works
The app built its login query by gluing the username directly into it. For the password check to mean anything, the database must read both halves of the AND. But administrator'-- comments out the password half before the database reads it. The lock is still on the door — I just deleted the part of the question that checks the key.
The fix: parameterized queries (prepared statements). The query structure is locked first, and the username is passed separately as pure data — never as part of the command. Then administrator'-- is treated as a literal username, no such user exists, and login fails.

How I'd spot this in a real app

Test the username field first — it's usually built into the query before the password.
Probe with a single ' to confirm the field is injectable (look for a 500 or broken response).
Inject administrator'-- (or admin'--). If -- fails, add a trailing space: administrator'-- .
Watch the response code — a 302 to an account page is success, even if your tool then bounces you (that's a cookie issue, not a failed attack).
The session cookie logs you in. In Repeater, carry the new Set-Cookie forward, or finish in the browser.
Don't stop at administrator — try other usernames, or ' OR 1=1 LIMIT 1-- to log in as the first user.

