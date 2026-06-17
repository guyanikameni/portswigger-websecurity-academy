Authentication bypass via information disclosure
Description
This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.
To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user carlos.
You can log in to your own account using the following credentials: wiener:peter

Solution
Step 1 — Log in with the given account.

I started by logging in as the wiener user at the login page.

<img width="1087" height="1603" alt="image" src="https://github.com/user-attachments/assets/3cd945b1-f5fe-48dd-884f-1e5b9f0d955c" />

Step 2 — Check the page source. Nothing useful.

I looked through the page source — no comments, no hidden hints worth following. So I went after the admin page directly by adding /admin to the URL.

I got this information disclosure message:

"Admin interface only available to local users"


<img width="1056" height="861" alt="image" src="https://github.com/user-attachments/assets/1d73d902-fcb9-4593-967f-44fb01028f06" />

This tells me the app decides access by where the request comes from — not who I am. So I started analyzing the request to understand more.

Step 3 — Change the request method.

I sent the request to Repeater and tried changing the HTTP method to see if the response changed. I used the TRACE method.

TRACE /admin HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net

The response came back with status code 200.

<img width="2167" height="852" alt="image" src="https://github.com/user-attachments/assets/0de977ae-773f-4ddd-8ded-04984c308eb5" />

Step 4 — Read the leaked header.

TRACE is a debug method — it echoes my own request back to me. And in that echo, the front-end had added an important header:

X-Custom-IP-Authorization: 37.101.171.137

This is the custom header the front-end uses to check if the request is local. The value is my own public IP.


Step 5 — Make Burp add the forged header to every request.
Forging it once in Repeater proves the header works. But to use the admin panel in my browser, I need that header on every request automatically.
I went to Burp Proxy → Match and replace → Add.

<img width="1614" height="869" alt="image" src="https://github.com/user-attachments/assets/e92f0e17-f917-459d-af1e-20c1ee27295c" />


I set the rule like this:

Type: Request header
Match: left empty (empty means "don't replace anything — just add a new header")
Replace: X-Custom-IP-Authorization: 127.0.0.1
Regex match: unticked

I clicked OK and made sure the rule was Enabled.
Now Burp adds X-Custom-IP-Authorization: 127.0.0.1 to every request leaving my browser. The app reads it, sees localhost, and treats me as a local user on every page.

After that, Step 6 (browse the home page, the Admin panel link now appears) and Step 7 (delete carlos) flow on naturally.



<img width="1092" height="849" alt="image" src="https://github.com/user-attachments/assets/ee7f57f6-299d-4a11-88c3-ede135b61168" />


<img width="1141" height="843" alt="image" src="https://github.com/user-attachments/assets/82b098d3-7c90-4fd5-8eb2-3b6b3aba7c65" />







