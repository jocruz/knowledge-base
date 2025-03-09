# JWT Authentication Bypass via JWK Header Injection

## **Lab: JWT Authentication Bypass via JWK Header Injection**

**Lab URL:** [https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection)

***

### **High-Level Summary**

**Objective:** Modify and sign a JWT token using the `jwk` header to gain unauthorized access to the admin panel and delete the user `carlos`.

#### **Key Techniques & Concepts**

* **JWT Structure:** Header, Payload, Signature
* **JWK Header Injection:** The server allows an attacker to embed an untrusted key directly in the JWT token for signature verification.
* **Token Manipulation:** Modify the `sub` claim and embed an attacker-controlled public key for privilege escalation.

#### **What Makes This Vulnerable?**

* The server fails to verify whether the provided public key (`jwk`) originated from a trusted source.

***

### **PortSwigger Solution**

#### **Step 1: Set Up Burp Suite**

* Load the **JWT Editor** extension from the **BApp Store**.

#### **Step 2: Log into the Lab**

* Use the provided credentials: `wiener:peter`.
* Capture the `GET /my-account` request using **Burp Suite** and send it to **Repeater**.

#### **Step 3: Modify the JWT Token**

* Change the request path from `/my-account` to `/admin`.
* Send the request and observe that access is denied.

#### **Step 4: Generate an RSA Key**

* Go to the **JWT Editor Keys** tab.
* Click **New RSA Key** > **Generate** > **OK**.

#### **Step 5: Modify the JWT Claims**

*   In the **JWT Tab** of Repeater, change the `sub` claim from:

    ```json
    "sub": "wiener"
    ```

    to:

    ```json
    "sub": "administrator"
    ```

#### **Step 6: Inject the JWK Key**

* Click the **Attack** button and choose **Embedded JWK**.
* Select the previously generated RSA key and click **OK**.

#### **Step 7: Review the JWT Header**

* Confirm that the `jwk` parameter containing the attacker-controlled public key has been added.

#### **Step 8: Send the Modified Request**

* Send the request and verify successful access to the admin panel.

#### **Step 9: Delete the User `carlos`**

* Locate the `/admin/delete?username=carlos` endpoint.
* Send the request and complete the lab.

***

### **Instructor Solution**

#### **Step 1: Log into the Web Application**

* Use the credentials `wiener:peter` and capture traffic using **Burp Suite**.

#### **Step 2: Identify the JWT Token**

* Locate the `GET /my-account` request containing a JWT token.
* Send the request to **Repeater**.
* Remove the `?id=wiener` parameter from the request URL.

#### **Step 3: Generate an RSA Key**

* Go to the **JWT Editor Keys** tab.
* Click **New RSA Key** and generate a key pair.

#### **Step 4: Modify the Token**

* In the **JWT Tab**:
  * Change the `sub` claim from `wiener` to `administrator`.
  * Click **Attack** > **Embedded JWK**.
  * Select the generated RSA key and confirm.

#### **Step 5: Send the Request**

* Send the modified request to `/admin` and verify successful access.

#### **Step 6: Apply the Token in the Browser**

* Copy the modified JWT from **Burp Repeater**.
* Go to the browser's **Developer Tools > Application > Cookies**.
* Replace the session cookie with the modified JWT.
* Change the URL to `/my-account` and press **Enter**.

#### **Step 7: Delete the User `carlos`**

* Go to `/admin/delete?username=carlos` to delete the user and complete the lab.

***

### **Explanation of Payloads and Techniques**

#### **JWT Structure**

A JWT consists of three parts separated by periods (`.`):

1. **Header:** Specifies the algorithm and key type.
2. **Payload:** Contains claims such as `sub` and `exp`.
3. **Signature:** Ensures token integrity and authenticity.

#### **Why This Attack Works**

* The server **trusts** the attacker’s key from the token instead of its own.
* Since the attacker **controls both the signature and the key**, they can generate a valid token pretending to be any user (such as `administrator`).

***

#### **Key Attack Variations**

* **Embedded JWK Injection:** Using an attacker-controlled key to sign tokens.
* **None Algorithm Attack:** Removing the signature entirely (from previous labs).

***

### **Tools and Commands**

* **Burp Suite:** Repeater, JWT Editor Extension, JWT Keys Tab
*   **Payload Example:**

    ```json
    {
      "sub": "administrator",
      "jwk": "<attacker-controlled public key>"
    }
    ```
* **Resources:** [PortSwigger JWT Cheat Sheet](https://portswigger.net/web-security/jwt)

