# JWT Authentication Bypass via Flawed Signature Verification (PortSwigger Lab)

## &#x20;Lab: JWT Authentication Bypass via Flawed Signature Verification

Lab URL: [https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)

***

### High-Level Summary

**Objective:** Modify a JWT token to gain unauthorized access to the admin panel and delete the user `carlos`.

#### **Key Techniques & Concepts:**

* **JWT Structure:** Header, Payload, Signature
* **Flawed Signature Verification:** The server is misconfigured to accept unsigned JWT tokens.
* **Token Manipulation:** Modify the `sub` claim and remove the signature for privilege escalation.

#### **What Makes This Vulnerable?**

* The server fails to verify the signature of the JWT token, allowing an attacker to remove the signature and modify claims.

***

### **PortSwigger Solution**

#### **Step 1: Log into the Lab**

* Use the provided credentials: `wiener:peter`.

#### **Step 2: Capture the JWT**

* In **Burp Suite**, go to **Proxy > HTTP history**.
* Locate the `GET /my-account` request after login.
* Observe the `session` cookie containing a JWT.

#### **Step 3: Decode the JWT**

* Use **Burp Inspector** to examine the token structure.
* Identify the claims inside the JWT (`sub` containing `wiener`).

#### **Step 4: Modify the Token**

* Send the request to **Repeater (Ctrl+R)**.
* Change the request path from `/my-account` to `/admin`.
* Modify the `sub` claim from `wiener` to `administrator` using the **JWT** tab.

#### **Step 5: Change the Signing Algorithm**

* Select the **JWT Header** in **Burp Inspector**.
* Change the `alg` value from `RS256` to `none`.
* Remove the signature but keep the trailing dot `.`.

#### **Step 6: Access the Admin Panel**

* Send the modified request and confirm access to the admin panel.

#### **Step 7: Delete the User `carlos`**

* Locate the `/admin/delete?username=carlos` endpoint.
* Send the request with the modified JWT to delete `carlos` and complete the lab.

***

### **Instructor Solution**

#### **Step 1: Log into the Web Application**

* Navigate to the `/login` page and log in with `wiener:peter`.
* Open **Burp Suite > Proxy > HTTP History**.

#### **Step 2: Inspect the JWT Token**

* Locate the `POST /login` request where a JWT token is issued.
* Examine the **JSON Web Token** components:
  * **Header:** Algorithm (`alg`: RS256)
  * **Payload:** Subject (`sub`: wiener)
  * **Signature:** Base64-encoded signature

#### **Step 3: Modify the JWT Claim**

* Send the `/my-account` request to **Repeater**.
*   Change the `sub` claim from:

    ```json
    "sub": "wiener"
    ```

    to:

    ```json
    "sub": "administrator"
    ```
* Click **Apply changes** in Burp.

#### **Step 4: Change the Signing Algorithm**

* Select the **JWT header** in **Burp Suite Inspector**.
* Change the `alg` value from `RS256` to `none`.
* Remove the signature manually while keeping the trailing dot `.`.

#### **Step 5: Using the Modified Token in the Browser**

* Copy the modified JWT token from **Repeater**.
* Open **Developer Tools > Application > Cookies** in the browser.
* Replace the existing `session` cookie with the modified JWT.

#### **Step 6: Access Admin Panel and Delete Carlos**

* Confirm access to the **Admin Panel** using the modified token.
* Navigate to `/admin/delete?username=carlos` and send the request.

#### **Step 7: Using the Attack Button in Burp Suite**

* The **Attack Button** in the **JWT Tab** automates:
  * Changing the `alg` claim to `none`.
  * Removing the signature portion of the token.
* This simplifies the attack process while testing.

✅ **Lab completed successfully.**

***

### **Explanation of Payloads and Techniques**

#### **JWT Structure**

A JSON Web Token (JWT) consists of three parts separated by periods (`.`):

1. **Header:** Specifies the signing algorithm (`RS256` by default).
2. **Payload:** Contains claims, such as `sub` and `exp`.
3. **Signature:** Ensures token integrity and authenticity.

#### **Why the Attack Works**

* The server is configured to accept tokens where the `alg` is set to `none`.
* By modifying the **JWT Header** and setting `alg: none`:
  * No signing key is required.
  * The token's integrity is no longer validated.
* Removing the signature and modifying the `sub` claim allows privilege escalation.

#### **Key Attack Variations**

* **Removing the Signature Manually:** Deleting the signature after the payload.
* **Changing the Algorithm to `none`:** Using the **Attack Button** in **Burp** for automation.

***

### **Tools and Commands**

* **Burp Suite:** Repeater, JWT Editor Extension, Attack Button
*   **Payload:**

    ```json
    {
      "sub": "administrator",
      "alg": "none"
    }
    ```
* **Resources:** [PortSwigger JWT Cheat Sheet](https://portswigger.net/web-security/jwt)

