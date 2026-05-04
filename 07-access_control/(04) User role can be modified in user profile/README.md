# Lab: User Role Can Be Modified in User Profile

This walkthrough covers the PortSwigger Web Security Academy lab: **User role can be modified in user profile**.

**Lab link:** https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile  
**Difficulty:** Apprentice  


## Known Information

From the lab description, we know the following:

- The application contains an admin panel at `/admin`.
- The admin panel is only accessible to users with `roleid=2`.
- We have valid user credentials: `wiener:peter`.
- The goal is to access the admin panel and delete the user `carlos`.

## Steps

### Analysis

Like in the previous lab, I start by exploring the shop website to understand how the application works.

The public shop pages do not reveal anything useful, so I log in using the provided credentials:

```text
wiener:peter
```


<img width="1106" height="537" alt="image" src="https://github.com/user-attachments/assets/4098de26-065d-4b28-b321-1f8e4511bbdd" />



### Intercepting the email update request

On the user profile page, there is an option to update the email address.

I change the email value and capture the request in **Burp Suite**. This allows me to inspect the data being sent from the browser to the server.

This step is important because profile update requests sometimes contain hidden values or accept extra parameters that are not shown in the form.

After sending the email update request, I also check the server response. The response looks interesting because it includes user information that may help identify how roles are handled by the application.


<img width="2241" height="989" alt="image" src="https://github.com/user-attachments/assets/f9263233-202a-4b6e-a90a-c155b70e87d2" />


The request contains the new email address in JSON format.

The response is interesting because the server echoes back what looks like the full account data. Instead of only returning the updated email, the response also includes other user information.

This is important because it may reveal internal fields used by the application, such as the user role.

### Modifying other account data

Next, I want to check whether the profile update function only changes the email address, or whether it accepts other fields sent in the JSON request.

To test this, I modify the intercepted request in Burp Suite and add an extra field to the JSON body.

If the server accepts this extra field, it may be possible to change account properties that a normal user should not be able to control.


<img width="2170" height="796" alt="image" src="https://github.com/user-attachments/assets/c7990969-fd54-4ad5-be26-680a60b72072" />


<img width="2163" height="837" alt="image" src="https://github.com/user-attachments/assets/3ec4763d-2f0a-4836-8858-cbeac0d82668" />




The server does not apply the modified `username` value, but it does apply the modified `roleid`.

This confirms that the application is vulnerable because it allows a normal user to modify their own role through the profile update request.

After setting my `roleid` to `2`, I can access the admin panel 


<img width="1087" height="764" alt="image" src="https://github.com/user-attachments/assets/f702e4af-63c5-48ef-9583-daa5ff99b37e" />




