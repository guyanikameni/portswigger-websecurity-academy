# Lab: User ID Controlled by Request Parameter

This walkthrough covers the PortSwigger Web Security Academy lab: **User ID controlled by request parameter**.

**Lab link:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter  
**Difficulty:** Apprentice  


## Known Information

From the lab description, we know the following:

- The application has a horizontal privilege escalation vulnerability on the user account page.
- We have valid user credentials: `wiener:peter`.
- The goal is to obtain and submit the API key of the user `carlos`.

## Steps

### Analysis

The first step is to explore the web application and understand how it works.

I do not expect the main shop pages to contain anything very interesting for this lab, so I log in using the provided credentials:

```text
wiener:peter
```


<img width="1054" height="592" alt="image" src="https://github.com/user-attachments/assets/adde617a-5a01-4f6f-a4d9-f50de0592f80" />




### Checking the Account Page

After logging in as `wiener`, I opened the account page.

The page displays the API key linked to my account. While reviewing the URL, I noticed that the application uses a request parameter to decide which user account should be displayed:

```http
/my-account?id=wiener
```



This is interesting because the application is allowing the user identity to be controlled from the URL.

In a secure application, the server should use the authenticated session to determine which account belongs to the logged-in user. A normal user should not be able to change a URL parameter and access another user’s data.

### Testing the User ID Parameter

To test for horizontal privilege escalation, I changed the value of the `id` parameter from my username to another known username:

```
/my-account?id=carlos
```

<img width="2506" height="729" alt="image" src="https://github.com/user-attachments/assets/cb503969-c779-4f26-8a49-1e355c85a96e" />



After sending the modified request, the application returned Carlos’s account page instead of blocking the request.

This confirms that the application does not properly verify whether the logged-in user is allowed to access the requested account.

As a result, I was able to view Carlos’s API key and submit it to solve the lab.



<img width="1590" height="926" alt="image" src="https://github.com/user-attachments/assets/a87ea2e4-7218-4d72-8fa1-51447a2fa74e" />
