# Lab: Unprotected Admin Functionality with Unpredictable URL

This walkthrough covers the PortSwigger Web Security Academy lab: **Unprotected admin functionality with unpredictable URL**.

**Lab link:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url  
**Difficulty:** Apprentice  


## Known Information

From the lab description, we know the following:

- The application contains an unprotected admin panel.
- The admin panel URL is not predictable.
- The URL is disclosed somewhere inside the application.
- The goal is to delete the user `carlos`.

## Steps

### Analysis

As usual, the first step is to analyze the website and understand how it works.

In this lab, the admin panel cannot be found through `robots.txt`. Also, because the URL is unpredictable, common directory discovery methods are unlikely to find it.

This means we need to look for the admin URL somewhere inside the application itself.

While reviewing the website, I noticed that the navigation links at the top of the page are generated dynamically. The application uses client-side code to insert an admin link for administrative users.

This is not a secure approach because anything sent to the browser can be viewed by the user. If the admin URL is included in the page source or JavaScript code, an attacker can find it even if the link is not visible on the page.


<img width="1707" height="755" alt="image" src="https://github.com/user-attachments/assets/865b6ec4-adba-4e25-9f72-0e2d19509175" />



Once the admin URL is discovered, solving the lab is straightforward.

I opened the exposed admin panel in the browser and clicked **Delete** next to the user `carlos`. This completed the lab.

<img width="2239" height="623" alt="image" src="https://github.com/user-attachments/assets/6bcd59b7-b743-4208-af1c-c4d13b143030" />




<img width="2202" height="736" alt="image" src="https://github.com/user-attachments/assets/0e38a027-5746-4610-94e6-4b8830346e47" />

