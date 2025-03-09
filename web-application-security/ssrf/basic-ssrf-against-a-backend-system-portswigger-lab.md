# Basic SSRF Against a Backend System (PortSwigger Lab)

## **Basic SSRF Against a Backend System – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **Server-Side Request Forgery (SSRF) vulnerability** where an attacker can force the application to make unauthorized requests to an **internal backend system**. The objective is to **scan an internal IP range (`192.168.0.X:8080`) for an admin interface**, locate the correct IP address, and exploit SSRF to **delete the user Carlos**.

* **Lab URL**: [PortSwigger Basic SSRF Against a Backend System Lab](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)
* **Target**: Identify an **internal admin panel** and perform an **unauthorized administrative action** using SSRF.

***

### **Analysis and Exploitation**

#### **1. Understanding the Vulnerability**

* **Why is this an SSRF issue?**
  * The application allows user-supplied input in the `stockApi` parameter.
  * The server **trusts user input** and makes requests to arbitrary URLs.
  * The attacker can use the server as a proxy to interact with internal services (`192.168.0.X:8080`).
* **Why is this critical?**
  * The vulnerability allows an attacker to **scan the internal network**, find admin panels, and **perform privileged actions** without direct access to the backend system.

***

#### **2. Exploiting the Vulnerability**

**Step 1: Intercept the Stock Check Request**

1. Navigate to a product page and initiate a **stock check** request.
2. Use **Burp Suite** to intercept the **POST request to /product/stock**.
3.  Identify the `stockApi` parameter containing a URL:

    ```plaintext
    stockApi=http://192.168.0.1:8080/product/stock/check?productId=1&storeId=1
    ```
4. Decode the URL in **Burp Decoder** to examine its full structure.

***

**Step 2: Scan for Internal Services**

1.  Modify the `stockApi` parameter to check for an **admin panel**:

    ```plaintext
    stockApi=http://192.168.0.1:8080/admin
    ```
2. Send the request in **Burp Repeater**.
3. If **no response or an error** occurs, try different IP addresses within the `192.168.0.X` range.

***

**Step 3: Automate Scanning Using Burp Intruder**

1. Highlight the **last octet of the IP address** (e.g., `1`) and mark it as a payload position.
2. Configure Burp Intruder:
   * **Payload Type**: Numbers
   * **Range**: 1–255
   * **Step**: 1
3. Start the attack and monitor responses:
   * **200 OK**: Possible admin interface detected.
   * **403 Forbidden** or **404 Not Found**: No access or no resource found.

***

**Step 4: Speed Up the Scan Using Turbo Intruder**

1. Install the **Turbo Intruder** extension in Burp Suite.
2.  Modify the script to automate requests for all IP addresses:

    ```python
    for i in range(1, 256):
        stockApi = 'http://192.168.0.%s:8080/admin' % i
        engine.queue(target.req, stockApi)
    ```
3.  Adjust the request format:

    ```plaintext
    stockApi=%s
    ```
4. Run the attack and **filter responses** by HTTP status codes to find the valid admin panel.

***

**Step 5: Locate the Delete User Endpoint**

1. Identify the **correct internal IP** hosting the admin panel.
2. Render the response in **Burp Suite’s Render tab**.
3.  Look for a **delete user link**, typically in an `<a>` tag:

    ```html
    <a href='http://192.168.0.236:8080/admin/delete?username=carlos'>
    ```

***

**Step 6: Exploit SSRF to Delete Carlos**

1.  Modify the `stockApi` parameter in **Burp Repeater**:

    ```plaintext
    stockApi=http://192.168.0.236:8080/admin/delete?username=carlos
    ```
2. Send the request and observe the response:
   * **302 Found**: The request was processed successfully.
3. Return to the lab interface and verify completion.

***

### **Security Breakdown: Why This is an SSRF Attack**

1. **User-Controlled Input Allows Arbitrary Requests**
   * The `stockApi` parameter enables user-defined requests.
   * Attackers can manipulate requests to access restricted internal resources.
2. **Accessing Internal Systems via SSRF**
   * The attacker forces the server to scan `192.168.0.X:8080/admin`.
   * The server acts as a proxy, enabling access to internal admin panels.
3. **Privilege Escalation and Unauthorized Actions**
   * The attacker exploits SSRF to interact with **admin-only endpoints**.
   * The delete request is executed **without authentication**, demonstrating improper access control.

***

### **Mitigation Strategies**

1. **Strict Input Validation**
   * Restrict the `stockApi` parameter to **trusted domains** using allowlists.
   * Reject user-supplied input containing **internal IPs** or sensitive endpoints.
2. **Block Internal Requests**
   * Configure network policies to **prevent servers from accessing internal resources** via external inputs.
   * Use **firewalls and access control lists** to block requests to `localhost` and internal IP ranges.
3. **Enforce Authentication for Internal Services**
   * Admin panels should require proper authentication and role validation.
   * Use session-based or token-based security mechanisms for sensitive actions.
4. **Monitor and Detect Suspicious Requests**
   * Log **all outgoing requests** made by the server.
   * Implement **rate limiting** to mitigate automated SSRF exploitation.

***

### **Final Thoughts**

This lab highlights how **SSRF can be exploited** to scan internal networks, discover hidden services, and perform **privileged actions**. Proper **input validation, network restrictions, and access controls** are essential to prevent such vulnerabilities in web applications.
