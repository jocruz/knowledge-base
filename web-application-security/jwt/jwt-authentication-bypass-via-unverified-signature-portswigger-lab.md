# JWT Authentication Bypass via Unverified Signature (PortSwigger Lab)

## Lab: JWT Authentication Bypass via Unverified Signature

### Lab URL

[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)

### High-Level Summary

**Objective:** Modify a JWT token to gain unauthorized access to the admin panel and delete the user `carlos`.

#### Key Techniques & Concepts

* **JWT Structure:** Header, Payload, Signature
* **Unverified Signature Flaw:** Server fails to validate the token signature.
* **Token Manipulation:** Modify the `sub` claim for privilege escalation.

#### What Makes This Vulnerable?

* The server does not verify the signature of the JWT tokens, allowing attackers to modify claims without proper validation.

***

### Steps to Exploit

#### Step 1: Log into the Web Application

*   Visit the `/login` page and log in with the credentials:

    ```plaintext
    Username: wiener  
    Password: peter  
    ```
* Navigate to **Burp Suite > Proxy > HTTP History**.

#### Step 2: Capture the JWT

* Look for the `GET /my-account` request after login.
* Identify the `session` cookie containing the JWT token.

#### Step 3: Decode the JWT

* Copy the token and use **Burp Inspector** or **jwt.io** to analyze its structure.
*   Identify the `sub` claim, which contains:

    ```json
    { "sub": "wiener" }
    ```

#### Step 4: Modify the JWT Claim

* Send the `/my-account` request to **Repeater**.
*   Change the `sub` claim from:

    ```json
    { "sub": "wiener" }
    ```

    to:

    ```json
    { "sub": "administrator" }
    ```
* Click **Apply changes** in Burp.

#### Step 5: Bypass Authentication

* Modify the request path from `/my-account` to `/admin`.
* Send the modified request and confirm access to the **Admin Panel**.

#### Step 6: Delete the User `carlos`

* Locate the `/admin/delete?username=carlos` endpoint.
* Send the request with the modified JWT to delete `carlos` and complete the lab.

***

### Explanation of Payloads and Techniques

#### **JWT Structure:**

A JSON Web Token (JWT) consists of three parts separated by periods (`.`):

1. **Header:** Specifies the algorithm used for signing the token (`RS256`).
2. **Payload:** Contains the claims, including `sub` (subject).
3. **Signature:** Verifies the authenticity of the token using a secret key.

#### **Why the Attack Works:**

* The server **fails to validate the signature**, so modifying the payload is sufficient to escalate privileges.
* By changing the `sub` claim to `administrator`, the server grants unauthorized access.

#### **Key Attack Variations:**

* **Removing the Signature:** Some misconfigured servers may not verify the signature at all.
* **Changing the Algorithm to** `none`: Switching the signing algorithm to `none` can bypass validation on vulnerable implementations.

***

### Tools and Commands

#### **Tools Used:**

* **Burp Suite:** Repeater, JWT Editor Extension
* **jwt.io**: For analyzing and modifying JWTs

#### **Payload:**

```json
{
  "sub": "administrator"
}
```

#### **Resources:**

* [PortSwigger JWT Cheat Sheet](https://portswigger.net/web-security/jwt)
