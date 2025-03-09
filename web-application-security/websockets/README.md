# WebSockets

## Introduction to Websockets

## **High-Level Summary**

**WebSockets** are a full-duplex communication protocol that enables **real-time, bidirectional** communication between a **client (browser)** and a **server** over a **single persistent TCP connection**. Unlike HTTP, which follows a **request-response model**, WebSockets maintain an **open connection** for continuous data exchange. They are commonly used in **live chat applications, multiplayer gaming, financial trading platforms, and real-time notifications**.

#### **Key Insights**

* WebSockets **reduce latency** by eliminating the overhead of repeated HTTP requests.
* They allow **event-driven communication**, meaning **clients and servers can send messages to each other without waiting**.
* **Security concerns** arise due to **lack of authentication on individual messages**, **input validation issues**, and **potential denial-of-service (DoS) attacks**.

***

### **Technical Definitions**

| **Term**                   | **Definition**                                                                           |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| **WebSocket**              | A persistent **full-duplex communication protocol** over a single TCP connection.        |
| **Full-Duplex**            | Both the client and server can send and receive messages **simultaneously**.             |
| **WebSocket Handshake**    | The initial HTTP **Upgrade request** that switches a connection from HTTP to WebSockets. |
| **TCP Connection**         | A two-way, persistent connection enabling WebSocket communication.                       |
| **WebSocket Frame**        | A structured data packet exchanged between a client and server.                          |
| **WSS (WebSocket Secure)** | The encrypted version of WebSockets using TLS (`wss://`).                                |
| **WebSocket API**          | The **JavaScript interface** used by browsers to interact with WebSockets.               |

***

### **How WebSockets Work**

1. **Handshake:** The client sends an **HTTP request** with an `Upgrade: websocket` header.
2. **Switching Protocols:** The server responds with `101 Switching Protocols`, establishing a WebSocket connection.
3. **Persistent Connection:** The connection remains open, allowing **bidirectional** communication.
4. **Data Exchange:** WebSocket **frames** (small packets) are used to send/receive data.
5. **Closing the Connection:** The client or server can terminate the connection using a `CLOSE` frame.

#### **Example: WebSocket Request and Response**

**Client Request (Handshake)**

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

**Server Response (Switching Protocols)**

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

***

### **WebSocket Example (JavaScript Implementation)**

```javascript
// Create a WebSocket connection
const socket = new WebSocket("wss://example.com/socket");

// Event listener for when the connection is opened
socket.onopen = () => {
    console.log("WebSocket connection established");
    socket.send("Hello Server!"); // Send data to server
};

// Event listener for receiving messages from the server
socket.onmessage = (event) => {
    console.log("Received:", event.data);
};

// Event listener for when the connection is closed
socket.onclose = () => console.log("WebSocket connection closed");
```

***

### **WebSockets vs HTTP**

| **Feature**             | **WebSockets**                       | **HTTP**                              |
| ----------------------- | ------------------------------------ | ------------------------------------- |
| **Communication Model** | Full-Duplex (Two-way)                | Request-Response (One-way)            |
| **Connection Type**     | Persistent (Open until closed)       | Stateless (Each request is new)       |
| **Latency**             | Low                                  | Higher due to new requests per action |
| **Use Cases**           | Real-time applications (Chat, Games) | Static pages, REST APIs               |

***

### **Identifying WebSockets in Penetration Testing**

#### **Methods for Discovery**

1. **Browser DevTools** (Network Tab → WS)
   * Open **Developer Tools** (`F12` or `Ctrl+Shift+I`).
   * Navigate to **Network > WS** to inspect WebSocket connections.
   * Observe **messages sent/received** in real-time.
2. **JavaScript Code Inspection**
   * Search for `new WebSocket("wss://...")` references in web applications.
3. **Checking HTTP Headers**
   * Look for **`Upgrade: websocket`** and **`Connection: Upgrade`** headers.
4. **Intercepting Traffic with Burp Suite**
   * Use **Burp Suite's WebSocket history** to analyze message payloads.
   * Test for **input validation issues** and **authentication bypass**.

***

### **Common WebSocket Security Vulnerabilities**

#### **1. Lack of Authentication on Individual Messages**

* WebSockets **authenticate the initial connection**, but messages within the session often lack verification.
* Attackers can **hijack an active WebSocket session** to send malicious requests.

**Example Exploit:**

```json
{"action": "update_balance", "amount": "999999"}
```

**Impact:** If **no session validation** occurs, an attacker can **manipulate financial transactions**.

**Remediation:**

* Ensure **each WebSocket message is authenticated** (e.g., attach **JWT tokens** to every request).
* Implement **server-side validation** to verify user actions.

***

#### **2. Input Validation Issues (XSS & SQL Injection)**

* **WebSocket messages often lack input sanitization**, making them vulnerable to:
  * **Cross-Site Scripting (XSS)**
  * **SQL Injection**
  * **Command Injection**

**Example Payload (XSS Injection)**:

```json
{"message": "<script>alert('Hacked!');</script>"}
```

**Impact:** If the server or another client processes the message **without sanitization**, XSS may execute.

**Remediation:**

* **Sanitize all incoming messages** before processing.
* Implement **Web Application Firewalls (WAFs)** that monitor WebSocket traffic.

***

#### **3. WebSocket Connection Hijacking**

* If **Cross-Site WebSocket Hijacking (CSWSH)** is possible, attackers can:
  * Trick a logged-in user into opening a malicious WebSocket connection.
  * Hijack the session and send unauthorized commands.

**Example Attack (CSWSH via Malicious Website):**

```javascript
const socket = new WebSocket("wss://victim.com/socket");
socket.onopen = () => {
    socket.send('{"action": "delete_account"}');
};
```

**Remediation:**

* **Enforce CORS policies** to restrict cross-origin WebSocket connections.
* Implement **authentication tokens** on **every WebSocket request**.

***

#### **4. Lack of Rate Limiting (DoS Attacks)**

* WebSockets **allow unlimited requests**, making them a target for **Denial of Service (DoS)** attacks.

**Example Attack:**

```javascript
while(true) {
    socket.send("Spam Request");
}
```

**Impact:** Attackers can **flood the server with requests**, leading to service degradation.

**Remediation:**

* Implement **rate limiting** on WebSocket connections.
* **Terminate inactive connections** after a period of inactivity.

***

### **WebSocket Security Best Practices**

#### **1. Always Use Secure WebSockets (`wss://`)**

* **Avoid `ws://` (unencrypted WebSockets)**, which expose traffic to **MITM attacks**.
* Use **TLS (`wss://`)** to protect data in transit.

#### **2. Authenticate Every WebSocket Message**

* Attach **JWT tokens** or **session-based authentication** to **every WebSocket message**.
*   Example:

    ```json
    {"token": "abc123", "action": "update_balance", "amount": "500"}
    ```

#### **3. Implement Server-Side Input Validation**

* Filter and sanitize all **WebSocket messages** to prevent **XSS and SQL injection**.

#### **4. Apply Rate Limiting**

* Restrict **message frequency per user** to prevent **WebSocket-based DoS attacks**.

#### **5. Enforce WebSocket CORS Policies**

* Restrict **WebSocket connections** to **trusted domains only**.

***

### **Penetration Testing Checklist for WebSockets**

* [ ] **Check for Authentication Issues:** Ensure messages require user authentication.
* [ ] **Test for Input Validation Vulnerabilities:** Inject **XSS/SQL payloads**.
* [ ] **Inspect CORS Policies:** Ensure WebSockets are **restricted to trusted origins**.
* [ ] **Monitor for DoS Attacks:** Test **rate limiting mechanisms**.
* [ ] **Analyze WebSocket Traffic:** Use **Burp Suite** to intercept and modify frames.

***

### **Additional Resources**

* [WebSockets RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455)
* [OWASP WebSockets Security Guide](https://owasp.org/www-community/WebSocket_Security)

***

### **Final Takeaways**

* **WebSockets offer low-latency, bidirectional communication**, but security risks exist.
* **Authenticate every message**—not just the initial handshake.
* **Implement rate limiting and proper input validation** to mitigate attacks.
* **Monitor WebSocket traffic** for unusual activity.
