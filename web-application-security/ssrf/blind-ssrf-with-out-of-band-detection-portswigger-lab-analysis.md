# Blind SSRF with Out-of-Band Detection (PortSwigger Lab Analysis)

## **Blind SSRF with Out-of-Band Detection – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates **Blind Server-Side Request Forgery (SSRF)**, where the application sends HTTP requests based on user-controlled input, but the attacker does not receive a direct response. Instead, out-of-band (OAST) detection methods such as **Burp Collaborator** or `webhook.site` are used to confirm the SSRF vulnerability.

* **Lab URL**: [PortSwigger Blind SSRF with Out-of-Band Detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)
* **Target**: Manipulate the **Referer header** to trigger a server-side request to an external server and confirm the vulnerability through out-of-band detection.

***

### **Analysis and Exploitation**

#### **1. Understanding the Vulnerability**

* **Why is this an SSRF issue?**
  * The application processes user-controlled input in the **Referer header** and makes an external request.
  * No validation is performed on the Referer header, allowing an attacker to force the server to request arbitrary external domains.
  * The attacker does not see the response, making this a **Blind SSRF** scenario.
* **Why is this critical?**
  * Attackers can force the server to interact with internal or external resources.
  * In real-world scenarios, this could lead to scanning internal networks, exfiltrating data, or even Remote Code Execution (RCE).

***

#### **2. Exploiting the Vulnerability**

**Step 1: Identify the Vulnerable Parameter**

1. Navigate to the lab and select a product.
2.  Use **Burp Suite** to intercept the **GET request** to the product page:

    ```html
    GET /product?productId=1 HTTP/2
    Host: <lab-url>
    Referer: https://<lab-url>/
    ```
3. The **Referer header** is used by the server, making it a potential injection point.

***

**Step 2: Modify the Referer Header**

1. Send the request to **Burp Repeater** (Ctrl+R).
2.  Modify the Referer header to another external domain:

    ```
    Referer: http://example.com/
    ```
3. Send the request and check if the response remains **HTTP 200 OK**.
4. If the request is accepted, it confirms that the application processes external domains **without validation**.

***

**Step 3: Insert Burp Collaborator Payload**

1. Open **Burp Collaborator** and generate a unique Collaborator URL.
2.  Modify the Referer header to point to the Collaborator URL:

    ```plaintext
    Referer: http://<unique-collaborator-url>
    ```
3. Send the request in **Burp Repeater**.

***

**Step 4: Confirm the SSRF via Out-of-Band Detection**

1. Navigate to the **Burp Collaborator** tab and click **Poll now**.
2.  Look for incoming **DNS and HTTP interactions**:

    ```plaintext
    Interaction type: DNS
    Hostname: <unique-collaborator-url>
    Interaction type: HTTP
    HTTP request to <unique-collaborator-url>
    ```
3. If an interaction appears, this confirms that the server made an outbound request, proving the **Blind SSRF vulnerability**.

***

### **Alternative Approach Without Burp Suite Pro**

If **Burp Suite Pro** is unavailable, use `webhook.site`:

1. Visit [webhook.site](https://webhook.site/) and generate a unique URL.
2.  Modify the Referer header to use the webhook URL:

    ```plaintext
    Referer: http://<unique-webhook-url>
    ```
3. Send the request and monitor the **webhook.site dashboard** for incoming requests.
4. A received request confirms the SSRF vulnerability.

***

### **Why This Solution is an Example of SSRF**

1. **User-Controlled Input Allows Arbitrary Requests**
   * The **Referer header** accepts external URLs, enabling the attacker to inject arbitrary endpoints.
2. **Accessing External and Internal Resources**
   * The attacker forces the server to make **unauthorized requests** on their behalf.
   * This could be extended to target **internal assets** or other restricted resources.
3. **Out-of-Band (OAST) Confirmation**
   * Since **Blind SSRF does not return a direct response**, tools like **Burp Collaborator** or `webhook.site` confirm the request execution.

***

### **Mitigation Strategies**

1. **Strict Input Validation**
   * Block user-controlled input in sensitive headers like **Referer**.
   * Use **allowlists** to permit only trusted domains.
2. **Restrict Internal Requests**
   * Implement **firewall rules** to prevent unauthorized outbound requests.
   * Block requests to internal services or non-public resources.
3. **Monitor and Log Server Requests**
   * Implement **logging** to detect unexpected external requests.
   * Use **rate-limiting** to prevent automated SSRF exploitation.

***

### **Final Thoughts**

This lab highlights how **Blind SSRF can be detected using out-of-band techniques**, even when no response is visible to the attacker. Proper **input validation, network restrictions, and monitoring** are crucial to mitigating this class of vulnerability.
