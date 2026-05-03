# Write-up: Unprotected admin functionality  PortSwigger Academy


<img width="737" height="209" alt="image" src="https://github.com/user-attachments/assets/97183985-97e3-445d-ac54-8b527a74adca" />


I will walk through the PortSwigger Web Security Academy lab **Unprotected admin functionality**.

This lab focuses on a common access control issue where an admin panel is exposed without proper protection. The goal is to identify the hidden admin functionality and understand why sensitive areas must never be left accessible without authorization checks.

**Learning path:** Server-side topics → Access control vulnerabilities  
**Lab link:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality  
**Difficulty:** Apprentice

## Lab description

<img width="550" height="99" alt="image" src="https://github.com/user-attachments/assets/0a6beaa8-f906-416f-8fa2-ceb9e46ad932" />




## Steps

The first thing I do is quickly explore the application to understand what kind of website I am dealing with. In this lab, the target is a simple shop website.

During the initial enumeration phase, one useful file to check is `robots.txt`.

This file is normally used to give instructions to search engine crawlers about which parts of the website should or should not be indexed. However, from a security testing point of view, it can sometimes reveal interesting paths that the developer did not want search engines to visit.

The important point is that `robots.txt` does not provide real protection. It only gives instructions to crawlers. Anyone can open it in the browser because it is just a plain text file.

So, as a tester, checking this file can help discover hidden or sensitive locations, such as admin panels or private directories.

There are different ways to check whether this file exists, but the simplest method is manual browsing.

### Manual browsing

I can try to access the file directly in the browser by visiting:

```http
/robots.txt
```


<img width="939" height="346" alt="image" src="https://github.com/user-attachments/assets/589e5df1-d2ad-488a-bb97-9589eea3af88" />



### Checking `robots.txt` with curl

Instead of opening the file in the browser, I can also check it directly from the terminal using `curl`:



<img width="1043" height="374" alt="image" src="https://github.com/user-attachments/assets/ecb4408f-1dbc-424d-9339-57636722bc9d" />


### Finding `robots.txt` with tools

Another way to discover `robots.txt` is by using a content discovery tool.

In most common wordlists, especially wordlists from **SecLists**, the `robots.txt` file is usually included. This means that during normal directory and file enumeration, tools like `gobuster` can quickly detect it.

For example, I can run `gobuster` against the target using a common web content wordlist:

gobuster dir -u https://0ae600510325336a8084b23c0014002a.web-security-academy.net/ -w /opt/Tool/SecLists/Discovery/Web-Content/common.txt



<img width="1707" height="755" alt="image" src="https://github.com/user-attachments/assets/56e8921c-d66e-475c-9779-f96e7cac24d1" />


### Automatic checks with Nikto

Some web scanners check for `robots.txt` automatically during enumeration.

For example, `Nikto` does not only detect the `robots.txt` file. It can also read the paths listed inside it and test whether those paths actually exist on the server.

This is useful because it saves time. Instead of manually opening every path found in `robots.txt`, Nikto reports the interesting entries directly in its output.


  nikto -h https://0ae600510325336a8084b23c0014002a.web-security-academy.net


<img width="2185" height="481" alt="image" src="https://github.com/user-attachments/assets/43ae6b94-7d03-40f4-a5ad-283805916d36" />




## Exploiting the issue

The key point in this lab is that `robots.txt` should never be used as a protection mechanism.

A `Disallow` entry only tells search engine crawlers which paths they should avoid indexing. It does not block access to those paths. Anyone can open the `robots.txt` file, read the hidden locations, and visit them manually.

In this case, the file revealed the following path:

http
/administrator-panel

To solve the lab, I opened the exposed admin panel directly in the browser:


<img width="2239" height="623" alt="image" src="https://github.com/user-attachments/assets/e2d19949-c339-414d-a6f2-92df9265f0fe" />




After discovering the hidden admin path, I visited `/administrator-panel` manually. Since the page was not protected, I was able to access the admin functionality and delete the user `carlos` to solve the lab.


<img width="2202" height="736" alt="image" src="https://github.com/user-attachments/assets/ada8aca0-070e-4ffd-b778-4f58498ea39f" />


