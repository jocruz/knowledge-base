# WebSocket Hijacking: Security Risks and Mitigation Strategies

## **WebSocket Hijacking: Security Risks and Mitigation Strategies**

***

### **High-Level Summary**

**WebSocket Hijacking** occurs when an attacker gains unauthorized access to an **active WebSocket connection**, allowing them to **intercept, manipulate, and send data** as the authenticated user. This attack is often executed using **Cross-Site WebSocket Hijacking (CSWSH)**, which exploits **poor authentication practices, cookie-based session management, and a lack of CSRF protections**.

#### **Key Risks**

* **Session Impersonation:** Attackers can **send malicious WebSocket messages** on behalf of a victim.
* **Data Theft:** Sensitive information transmitted over WebSockets can be **intercepted and exfiltrated**.
* **Command Injection:** If WebSocket messages control **application behavior**, hijackers may **execute unauthorized actions**.

***

### **Technical Definitions**

| **Term**                                   | **Definition**                                                                                  |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| **WebSocket Hijacking**                    | Unauthorized **access and control** of a WebSocket session.                                     |
| **Cross-Site WebSocket Hijacking (CSWSH)** | An attack that abuses **cookie-based authentication** to hijack an active WebSocket session.    |
| **CSRF (Cross-Site Request Forgery)**      | An attack where a victim unknowingly **performs an action** on a site where they are logged in. |
| **WebSocket Handshake**                    | The **initial HTTP request** that establishes a WebSocket connection.                           |
| **Session Token**                          | A **unique identifier** used for authenticating a user's session.                               |
| **Cookie-Based Authentication**            | A method where **cookies** store session credentials for authentication.                        |
| **Same-Origin Policy (SOP)**               | A security mechanism that **prevents cross-origin web requests** without explicit permission.   |

***

### **How WebSocket Hijacking Works (Step-by-Step)**

#### **Step 1: User Logs Into a Legitimate Web Application**

* The **user logs into a website** and receives a **session token** stored as a cookie.
* The WebSocket connection is established **using cookies** instead of token-based authentication.

#### **Step 2: Victim Visits a Malicious Website**

* The attacker **tricks the victim** into visiting a **malicious webpage**.
* The victim’s **browser automatically sends session cookies** when the malicious site initiates a WebSocket connection.

#### **Step 3: WebSocket Handshake Exploitation**

* The attacker's website **initiates a WebSocket handshake** with the legitimate server.
* If the server does not verify the origin or require authentication, it **accepts the WebSocket request**.

#### **Step 4: Hijacking the WebSocket Connection**

* The attacker can now **send and receive messages** over the victim’s WebSocket session.
* Depending on the application, the attacker can:
  * **Steal sensitive information** exchanged over WebSockets.
  * **Execute unauthorized commands** (e.g., fund transfers, account deletion).
  * **Extract authentication tokens** transmitted via WebSockets.

***

### **Example Attack Scenario**

#### **Vulnerable WebSocket Implementation (Using Cookie-Based Authentication)**

```javascript
const socket = new WebSocket("wss://bank.example.com/socket");

socket.onopen = () => {
    socket.send(JSON.stringify({ action: "view_balance" }));
};

socket.onmessage = (event) => {
    console.log("Balance:", event.data);
};
```

**Security Issue:**

* The **WebSocket connection relies solely on session cookies** for authentication.
* If a victim is **logged in** and visits a malicious page, **attackers can initiate a WebSocket session on their behalf**.

***

### **How to Identify WebSocket Hijacking Vulnerabilities**

#### **1. Analyze the WebSocket Handshake**

* **Intercept the WebSocket handshake request** using **Burp Suite** or **browser DevTools**.
* Look for **session-based authentication without additional security checks**.

#### **2. Check for Missing CSRF Protection**

* WebSockets should **validate an anti-CSRF token** before establishing a session.
* If CSRF tokens are absent, **attackers can reuse session cookies** to hijack connections.

#### **3. Inspect Origin Headers and CORS Policies**

* If the WebSocket **accepts requests from any origin**, **cross-site hijacking is possible**.
* Verify that **the WebSocket server only accepts connections from trusted origins**.

#### **4. Test for Token Reuse Vulnerabilities**

* Ensure that **WebSocket authentication tokens** are **not reusable across multiple connections**.
* Expired tokens should **not be accepted** for authentication.

***

### **Real-World WebSocket Hijacking Examples**

#### **1. Cross-Site WebSocket Hijacking in Slack (CVE-2020-XXXX)**

* In 2020, security researchers discovered that **Slack’s WebSocket API** allowed attackers to hijack a user’s session.
* **Impact:** Attackers could **impersonate victims**, send messages, and **modify Slack channels**.
* **Reference:** [CVE-2020-XXXX](https://nvd.nist.gov/)

#### **2. WebSocket Hijacking in Financial Trading Platforms**

* Several online **trading platforms** were found to have **weak WebSocket authentication**, allowing attackers to:
  * **Monitor market trades** in real-time.
  * **Spoof trade transactions** by manipulating WebSocket messages.
* **Reference:** [Security Research Paper](https://www.blackhat.com/docs/us-20/wednesday/US-20-Masson-WebSocket-Hijacking-Attacks.pdf)

#### **3. WebSocket Hijacking via Misconfigured Load Balancers (CVE-2021-XXXX)**

* Attackers exploited **misconfigured WebSocket load balancers** to **intercept WebSocket traffic**.
* **Reference:** [CVE-2021-XXXX](https://nvd.nist.gov/)

***

### **How to Secure WebSockets Against Hijacking**

#### **1. Use Token-Based Authentication Instead of Cookies**

* WebSockets should require **JWT tokens** or **API keys** for authentication.
* Example Secure WebSocket Connection:

```javascript
const socket = new WebSocket("wss://secure.example.com/socket?token=eyJhbGciOiJI...");
```

**Why?**

* The session token **must be explicitly sent** (not stored in cookies).
* Even if an attacker **hijacks a session**, they **cannot initiate a new WebSocket connection** without a valid token.

#### **2. Implement CSRF Protection on WebSocket Handshakes**

* Include **anti-CSRF tokens** in **handshake requests**.
* Verify the **CSRF token server-side** before upgrading to a WebSocket connection.

#### **3. Enforce Strict Origin Policies**

* Configure WebSockets to **only accept connections from trusted origins**.
* Example **Server-Side WebSocket Origin Validation (Node.js)**:

```javascript
const allowedOrigins = ["https://example.com"];

server.on("connection", (ws, req) => {
    if (!allowedOrigins.includes(req.headers.origin)) {
        ws.terminate(); // Reject connection if the origin is untrusted
    }
});
```

#### **4. Use Secure WebSockets (`wss://`)**

* Always enforce **TLS encryption** (`wss://`) to protect against **Man-in-the-Middle (MITM) attacks**.

#### **5. Implement Rate Limiting for WebSockets**

* Restrict excessive WebSocket connections to **prevent DoS attacks**.
* Example Rate Limiting Middleware:

```javascript
const rateLimit = require("express-rate-limit");

const wsLimiter = rateLimit({
    windowMs: 10 * 1000, // 10 seconds
    max: 5, // Limit each IP to 5 requests per window
});

app.use(wsLimiter);
```

***

### **Penetration Testing Checklist for WebSocket Security**

* [ ] **Check for Cookie-Based Authentication:** Ensure WebSockets require explicit authentication tokens.
* [ ] **Inspect WebSocket Handshake:** Verify that handshake requests include CSRF protection.
* [ ] **Analyze Origin Headers:** Confirm that WebSockets reject **cross-origin** connections.
* [ ] **Test for Token Reuse:** Ensure session tokens are **not valid across multiple connections**.
* [ ] **Validate Input Sanitization:** Inject **malicious WebSocket messages** to test for **XSS and SQL Injection**.

***

### **Additional Resources**

* [OWASP WebSockets Security Guide](https://owasp.org/www-community/WebSocket_Security)
* [RFC 6455 - The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455)
* [PortSwigger WebSocket Security Testing](https://portswigger.net/web-security/websockets)

***

### **Final Takeaways**

* **WebSocket hijacking is a critical security risk** when authentication is misconfigured.
* **Avoid cookie-based authentication**—use **JWT tokens** or **API keys**.
* **Enforce CSRF protection and origin checks** to mitigate cross-site attacks.
* **Monitor WebSocket activity** to detect **anomalous connections and messages**.
