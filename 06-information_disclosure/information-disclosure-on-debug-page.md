# Information disclosure on debug page

**Lab:** PortSwigger Web Security Academy — Information Disclosure (Apprentice)

---

## Description

This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the `SECRET_KEY` environment variable.

---

## Solution

<img width="1389" height="1811" alt="image" src="https://github.com/user-attachments/assets/b2b39f10-2353-472d-8e17-2f4084d52b9b" />

Looking into the page source (`CTRL+U`) and reading through the comments, we found this info disclosure, with a link to this config website:

<img width="1044" height="938" alt="image" src="https://github.com/user-attachments/assets/1f2766e1-1db0-4bec-8666-376494a0a88d" />




/cgi-bin/phpinfo.php

we try to open that on there browser  We  obtain info about the PHP version and all other configuration data — in our case, the `SECRET_KEY

https://0a6a006604924a3a886f7af0002c0058.web-security-academy.net/cgi-bin/phpinfo.php

<img width="1036" height="976" alt="image" src="https://github.com/user-attachments/assets/62e0a609-01f4-4ab0-a422-29536152ad17" />

The page is large, so by pressing `CTRL+F` and searching for the word `SECRET_KEY`, we found the flag sitting in plain text with the other configuration data.
