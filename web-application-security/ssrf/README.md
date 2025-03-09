# SSRF

## **Server-Side Request Forgery (SSRF) – Comprehensive Overview**

***

### **What is SSRF?**

Server-Side Request Forgery (SSRF) is a **web security vulnerability** where an attacker **manipulates a server-side application** to make unauthorized requests on their behalf. This often results in access to **internal resources**, which are otherwise inaccessible. SSRF attacks are **increasingly common** and can lead to:

* **Data breaches**
* **Unauthorized access to internal services**
* **Exploitation of cloud environments**
* **Remote code execution (RCE)**

***

### **Key Takeaways**

#### **1. How SSRF Works**

**Legitimate Process**

1. A web application allows users to **submit a URL** for processing (e.g., metadata fetching, stock availability checks, or image rendering).
2. The **server makes an outbound request** to the specified URL and returns the response to the user.

**Exploitation Path**

Attackers manipulate user-controlled input to force the **server** to interact with **internal systems**, such as:

* **Internal Admin Panels:** `http://localhost/admin`
* **Cloud Metadata APIs:** `http://169.254.169.254/latest/meta-data/` (AWS)
* **Internal APIs:** `http://internal.api.local/admin`

By exploiting SSRF, an attacker can **bypass firewall restrictions** and access **sensitive internal data**.

***

### **2. Common SSRF Attack Scenarios**

#### **Accessing Internal Services**

* Attackers interact with **internal-only services** that should not be externally accessible.
*   **Example:** Extracting **cloud credentials** from AWS metadata:

    ```
    http://169.254.169.254/latest/meta-data/iam/security-credentials/
    ```

#### **Port Scanning via SSRF**

* If the application accepts arbitrary URLs, **attackers can scan internal IP ranges** to detect open ports.
*   **Example:** Identifying **SSH service** running on an internal server:

    ```
    http://internal.local:22
    ```

#### **SSRF Leading to Remote Code Execution (RCE)**

* If an **internal API** allows **command execution**, SSRF can be used to **execute arbitrary code**.
* **Example:** Exploiting an SSRF vulnerability to send a request to an internal API that runs system commands.

***

### **3. Real-World SSRF Exploits & Incidents**

#### **CVE-2021-21985: VMware vCenter SSRF to Remote Code Execution**

* **Issue:** VMware vCenter contained an SSRF flaw that allowed attackers to access **internal management APIs**.
* **Exploit:** Attackers used SSRF to interact with the internal API, leading to **remote code execution (RCE)**.
* **Impact:** Gave attackers **full control** over VMware vCenter environments.
* **Reference:** [CVE-2021-21985](https://nvd.nist.gov/vuln/detail/CVE-2021-21985)

***

#### **Capital One Data Breach (2019) – AWS SSRF Exploitation**

* **Issue:** A former Amazon Web Services (AWS) employee exploited an **SSRF vulnerability** to access **Capital One’s internal cloud environment**.
* **Exploit:** The attacker leveraged SSRF to access **AWS metadata services**, extracting credentials that granted access to **sensitive customer data**.
* **Impact:** Exposed **over 100 million customer records**.
* **Reference:** [Capital One SSRF Breach](https://www.forbes.com/sites/kateoflahertyuk/2019/07/30/capital-one-hack-ssrf-attack-explained/)

***

### **4. Security Risks & Impact**

| **Risk**                        | **Impact**                                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Internal Data Exposure**      | Attackers can access **sensitive internal APIs**, cloud metadata, and credentials.                  |
| **Firewall Bypass**             | SSRF allows attackers to interact with **internal network resources**, bypassing security controls. |
| **Lateral Movement**            | Attackers can use SSRF to pivot within an internal network and escalate access.                     |
| **Remote Code Execution (RCE)** | If an internal API is vulnerable, SSRF can escalate to **full system compromise**.                  |

***

### **5. Preventing SSRF**

#### **1. Input Validation and Allowlisting**

* **Reject user-supplied URLs** unless they match a **predefined allowlist**.
* **Block access to private IP ranges**, such as:
  * `127.0.0.1` (localhost)
  * `10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16` (internal networks)

#### **2. Restrict Server-Side Network Requests**

* **Limit outgoing network requests** to **trusted services** only.
*   **Block access to cloud metadata APIs**, such as AWS’s:

    ```
    http://169.254.169.254/latest/meta-data/
    ```

#### **3. Monitor and Log Requests**

* Implement **logging and alerting** for requests targeting **internal services**.
* **Rate-limit** requests to prevent **automated scanning**.

#### **4. Use a Web Application Firewall (WAF)**

* Deploy **security rules** to detect and block **SSRF attempts**.

***

### **6. Secure Code Example – Preventing SSRF in Node.js**

```javascript
const allowedDomains = ['https://trusted-api.com'];

app.post('/fetch', async (req, res) => {
    const userURL = req.body.url;
    
    try {
        const parsedURL = new URL(userURL);
        
        // Allow only predefined trusted domains
        if (!allowedDomains.includes(parsedURL.origin)) {
            return res.status(403).send("Unauthorized request.");
        }

        // Perform request securely
        const response = await fetch(userURL);
        const data = await response.text();
        
        res.send(data);
    } catch (error) {
        res.status(400).send("Invalid URL.");
    }
});
```

#### **Why is this Secure?**

✅ **Allowlist Enforcement:** Only **trusted domains** are permitted.\
✅ **Rejects Arbitrary URLs:** Prevents user-controlled URLs from reaching internal services.\
✅ **Proper Error Handling:** Invalid URLs do not crash the application.

***

### **7. Testing for SSRF Vulnerabilities**

#### **1. Identify SSRF Entry Points**

* Look for **user-controlled URL inputs** in:
  * **Image upload previews**
  * **URL-fetching mechanisms** (e.g., metadata scrapers)
  * **Webhooks or API integrations**

#### **2. Modify Input to Access Internal Resources**

*   Replace external URLs with internal addresses, such as:

    ```
    http://localhost/admin
    http://169.254.169.254/latest/meta-data/
    ```

#### **3. Analyze Server Behavior**

* If the **application returns internal data**, it is vulnerable.
* If a request **takes longer than expected**, it may indicate an **open internal service** being probed.

***

### **8. References & Further Reading**

* **OWASP SSRF Cheat Sheet:** [OWASP Guide](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
* **CVE-2021-21985 – VMware vCenter SSRF to RCE:** [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2021-21985)
* **Capital One SSRF Breach (2019):** [Forbes Report](https://www.forbes.com/sites/kateoflahertyuk/2019/07/30/capital-one-hack-ssrf-attack-explained/)

***

### **Final Thoughts**

**SSRF vulnerabilities are a critical security concern** that can expose **internal systems, sensitive data, and cloud environments**. Proper mitigation requires **strict input validation, network restrictions, and continuous monitoring**. Organizations must **securely handle server-side requests** to prevent SSRF exploitation.
