# Basic SSRF Against Localhost (PortSwigger)

## **Basic SSRF Against Localhost – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **basic Server-Side Request Forgery (SSRF) vulnerability** where an attacker can manipulate a server-side request to interact with **internal services**. The goal is to **exploit the stock check feature** to access the admin interface on `localhost` and delete the user Carlos.

* **Lab URL**: [PortSwigger Basic SSRF Against Localhost Lab](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost)
* **Target:** Modify the `stockApi` parameter to make **unauthorized requests to the internal admin panel** and execute an **admin-level action**.

***

### **Analysis and Exploitation**

#### **1. Understanding the Vulnerability**

* **Why is this an SSRF issue?**
  * The application accepts **user-supplied URLs** in the `stockApi` parameter.
  * The server makes an outbound request to the specified URL without validation.
  * This allows an attacker to **manipulate the request to access internal resources** such as `http://localhost/admin`.
* **Why is this critical?**
  * The vulnerability enables an **unauthenticated user to interact with internal resources**, bypassing client-side security.
  * Attackers can **execute administrative actions** by forcing the server to send requests to internal endpoints.

***

#### **2. Exploiting the Vulnerability**

**Step 1: Intercept the Stock Check Request**

1. Navigate to the **Products** page and select a store using the stock check feature.
2. Open **Burp Suite** and locate the **POST request to /product/stock** in the HTTP history.
3.  Identify the **stockApi** parameter, which contains an encoded URL:

    ```plaintext
    stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
    ```
4.  **Decode the URL** in **Burp Decoder** to view its actual contents:

    ```plaintext
    http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
    ```

***

**Step 2: Verify SSRF with an External Service**

1. Use **Burp Collaborator** or an external tool like `webhook.site` to confirm SSRF.
2.  Copy a **unique Collaborator URL** and replace the `stockApi` value:

    ```plaintext
    stockApi=http://<your-collaborator-url>
    ```
3. Encode the modified URL in Burp Suite (`Ctrl+U`).
4. Send the request and check Collaborator for incoming server-side requests.

***

**Step 3: Access the Internal Admin Panel**

1.  Modify the **stockApi** parameter to request the **admin panel**:

    ```plaintext
    stockApi=http%3A%2F%2Flocalhost%2Fadmin
    ```
2. Send the request in **Burp Repeater**.
3. Observe the **HTTP response**, which reveals the admin interface.

***

**Step 4: Identify the Admin Delete Endpoint**

1. Inspect the response for **HTML elements** related to user management.
2.  Locate the **delete action for Carlos**, which appears as:

    ```plaintext
    /admin/delete?username=carlos
    ```

***

**Step 5: Delete Carlos via SSRF**

1.  Modify the **stockApi** parameter to send a request to the delete URL:

    ```plaintext
    stockApi=http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos
    ```
2. Send the request in **Burp Repeater**.
3. Observe the **HTTP/2 302 Found** response, confirming the request was processed.

***

**Step 6: Confirm Success**

1. Return to the **lab interface** and verify that Carlos has been deleted.
2. Alternatively, resend the request and check the **Render** tab in Burp Suite for the completed lab message.

***

### **Security Breakdown: Why This is an SSRF Attack**

1. **User-Supplied Input Controls the Server’s Requests**
   * The `stockApi` parameter **accepts full URLs** and makes **server-side requests** without validation.
   * Attackers can redirect requests to **internal or unauthorized resources**.
2. **Accessing Internal Systems**
   * The attacker forces the server to request `http://localhost/admin`, **bypassing access controls**.
   * The admin panel is **not directly accessible from the internet**, but the SSRF allows an external user to reach it.
3. **Privilege Escalation via SSRF**
   * The attacker interacts with the **admin delete endpoint**, **executing an admin-only action** without authentication.
   * By modifying the `stockApi` parameter, an unauthenticated user **performs an internal admin action**.

***

### **Mitigation Strategies**

1. **Implement Input Validation and Allowlisting**
   * Restrict the `stockApi` parameter to **only allow trusted domains**.
   * Use **regular expressions** to prevent attackers from supplying arbitrary URLs.
2. **Block Requests to Internal IPs and Hostnames**
   * Disallow requests to `localhost`, `127.0.0.1`, or private network ranges (`10.0.0.0/8`, `192.168.0.0/16`).
   * Use **firewall rules** to prevent **internal network access** from external requests.
3. **Enforce Role-Based Access Control (RBAC)**
   * Ensure sensitive endpoints **require proper authentication and authorization**.
   * The delete user functionality should be **restricted to verified admin users only**.
4. **Monitor and Log SSRF Attempts**
   * Log **all outgoing requests** from the server to detect unusual patterns.
   * Implement **rate limiting** to prevent **automated SSRF exploitation attempts**.

***

### **Final Thoughts**

This lab highlights how **SSRF can be exploited** to gain access to **internal resources** and perform **unauthorized actions**. Proper security controls, such as **input validation, allowlisting, and network restrictions**, are critical to **prevent SSRF vulnerabilities** in modern web applications.
