# SSRF

## **Server-Side Request Forgery (SSRF) – Overview**

**What is SSRF?**

Server-Side Request Forgery (SSRF) occurs when an attacker manipulates a server-side application into making unauthorized requests. This often results in the exposure of internal resources that are otherwise inaccessible. SSRF vulnerabilities are increasingly common and can lead to **data breaches, unauthorized access to internal services, or even remote code execution (RCE).**

***

### **Key Takeaways**

#### **1. How SSRF Works**

* **Legitimate Process**
  * A web application allows users to submit a URL for processing (e.g., fetching metadata, stock availability checks, or image rendering).
  * The server makes an outbound request to the specified URL and returns the response to the user.
* **Exploitation Path**
  * Attackers modify the input to access internal systems, such as:
    * **Internal Admin Panels:** `http://localhost/admin`
    * **Metadata APIs:** `http://169.254.169.254/latest/meta-data/` (AWS instance metadata)
    * **Internal API endpoints:** `http://internal.api.local/admin`
  * This manipulation forces the server to **bypass firewall restrictions** and retrieve unauthorized information.

***

#### **2. Common SSRF Attack Scenarios**

* **Accessing Internal Services**
  * Attackers interact with **internal-only** services that should not be externally accessible.
  * Example: Accessing internal databases or cloud metadata services to extract credentials.
* **Port Scanning via SSRF**
  * If the application allows flexible URL inputs, attackers can scan internal IP ranges to identify open services.
  * Example: `http://internal.local:22` (SSH service detection).
* **SSRF Leading to Remote Code Execution (RCE)**
  * If an internal service processes user input unsafely, SSRF can escalate to RCE.
  * Example: Exploiting an SSRF vulnerability to reach an internal API that executes system commands.

***

### **Security Risks and Impact**

#### **1. Internal Data Exposure**

* SSRF allows attackers to access internal assets **not meant for public exposure**, such as:
  * Internal APIs
  * Cloud metadata services (e.g., AWS, GCP, Azure)
  * Configuration files or credentials

#### **2. Firewall Bypass**

* The server acts as a **proxy**, allowing attackers to interact with internal network resources that would normally be protected.

#### **3. Lateral Movement**

* Attackers can chain SSRF with other vulnerabilities to **pivot deeper into an internal network** and escalate access.

***

### **Preventing SSRF**

#### **1. Input Validation and Allowlisting**

* Only allow **predefined, trusted URLs** instead of accepting user-inputted URLs.
* Reject requests containing private IP ranges, such as:
  * `127.0.0.1` (localhost)
  * `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` (internal subnets)

#### **2. Restrict Server-Side Network Requests**

* Limit outgoing network requests **only to trusted services**.
* Block unnecessary access to cloud metadata APIs, such as AWS’s `http://169.254.169.254`.

#### **3. Monitor and Log Requests**

* Implement **logging and alerting** for unusual requests, particularly those targeting internal services.
* Rate-limit requests to prevent **automated scanning attempts**.

#### **4. Use a Web Application Firewall (WAF)**

* Deploy security rules to detect and block **malicious SSRF attempts** in HTTP requests.

***

### **Testing for SSRF Vulnerabilities**

#### **1. Identify SSRF Entry Points**

* Look for **URL-based user inputs** in:
  * Image upload or preview functionality
  * URL fetching mechanisms (e.g., metadata scrapers)
  * Webhooks or API integrations

#### **2. Modify Input to Access Internal Resources**

* Replace external URLs with internal addresses, such as:
  * `http://localhost/admin`
  * `http://169.254.169.254/latest/meta-data/`

#### **3. Analyze Server Behavior**

* If the application returns unexpected internal data, it is vulnerable.
* If the request takes longer than usual, it might indicate an **open port being probed**.

***

### **Final Thoughts**

SSRF vulnerabilities are a **critical security concern** that can expose **internal systems, sensitive data, and cloud environments**. Proper mitigation requires **strict input validation, network restrictions, and continuous monitoring** to prevent unauthorized internal access. Organizations must ensure **server-side request handling is secure** to prevent SSRF exploitation.
