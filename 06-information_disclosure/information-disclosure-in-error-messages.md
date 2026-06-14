Information disclosure in error messages
Description

Information disclosure in error messages
Description

This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.
Solution

Checking into page source code there're not of interesting, so click to one of products shop: https://0a6c00fe03599c7f8a11a3f700100029.web-security-academy.net/product?productId=1

the idea is to generate and error, than we try to inject something with a SQLi:

and obtaining an error by Apache Struts 2 2.3.31 we've discovered the vs number of this framework.

Solution



<img width="1083" height="1610" alt="image" src="https://github.com/user-attachments/assets/b7df2be5-0fbd-4e65-88df-22647d1ad29e" />



First I checked the page source code. (Source code = the raw HTML behind the page, viewable in the browser.) Nothing interesting there.

The idea is simple: make the site crash on purpose. A crash often spills useful information.
I tried injecting something into the productId value — a SQL injection style payload. (SQL injection = sending database commands into an input to confuse the app.) The point wasn't to break the database. It was just to feed the app something it didn't expect and force an error.



<img width="2206" height="993" alt="image" src="https://github.com/user-attachments/assets/490dd4f2-adda-4a41-9052-a046c4400ab3" />



The app choked and threw an error. And that error told me everything:
Apache Struts 2 — version 2 2.3.31
[INSERT SCREENSHOT: the error message showing Apache Struts 2 2.3.31]
That's the framework version. I submitted 2 2.3.31 and solved the lab.

What to note down:

Source code first. Quick check, sometimes free wins.
No luck there? Make the app crash on purpose.
Trick: inject an unexpected value (SQLi-style, or just text) into a parameter that expects a number.
The crash report (stack trace) often names the framework + exact version.
Old version = known public exploits. That's why this matters.
Lab answer string: exactly 2 2.3.31.

One-line summary: Force a crash on a product parameter, read the error, steal the framework version — 2 2.3.31.
