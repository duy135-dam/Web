# Revoked 

The goal is to access the restricted `/admin` panel to retrieve the flag. The application requires authentication (via JWT) to view the employee list, and administrative privileges are enforced on the backend to view the flag.
## JWT attack part via Token Forgery 
upon creating an account and authenticating we are given a JWT for our session

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/cookie.png)

 this same token when decoded gives the following : 
 
![phoyo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/jwt.png)
The first idea that comes to mind is : changing the algorithm to "none" and the is_admin to 1 in order to become admin and get to the secret room where the flag is being held 
but this will not work because of the following code on the backend :

**Algorithm Validation:** The `jwt.decode` function explicitly enforces `algorithms=["HS256"]`, preventing "None" algorithm attacks.

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/code.png)

**Privilege escalation (Token Forgery):** Even if we could forge a token with `"is_admin": 1`, the application does not trust the token's claim for authorization.

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/code2.png)

The application validates the user's role by querying the database using the username inside the token. Therefore, we must compromise the database to impersonate an admin.

## Identifying the vulnerability 

After digging deep in the application's code We can locate a SQL Injection vulnerability in the `/employees` search endpoint and find this particular vulnerable line of code :
![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/code3.png)
The application uses an **f-string** to insert user input (`query`) directly into the SQL statement. This allows for Union-Based SQL Injection.

So we can use this technique to leak data in other tables such as users and revoked tokens 
![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/code5.png)
## Exploitation 

### Step 1: Determining Column Count

To perform a Union-Based injection, we must match the column count of the original query (`id`, `name`, `email`, `position` = 4 columns). We injected a payload to visualize where our data would appear.

payload : `' UNION SELECT 1,2,3,4 --`

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/columns.png)

And as we can see only the 2nd and 4th column are being reflected 

### Step 2: Database Enumeration (Recon)
Using this information we can get the ID of the admin account from the users table

payload : `' UNION SELECT 1,is_admin,3,id FROM users --` 

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/mapping.png)
and from here we can see that the ID 1 and 2 are admin 

### Final step get the Tokens : 

payload : `' UNION SELECT 1, token, 3, id FROM revoked_tokens --` 

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/tokens.png)


## Post-Exploitation 

by changing the token to any of the tokens corresponding to ID=1 or ID=2 we become admin user and we finally can navigate to the Admin panel 

![photo](https://github.com/Naiderr/CTF-Writeups/blob/main/images/win.png)


and so here is the flag for completing the challenge : `Hero{N0t_th4t_r3v0k3d_ec6dcf0ae6ae239c4d630b2f5ccb51bb}`


