# XSS Methodology

## XSS Methodology

### 1. Discovery and Mapping

#### Identifying Potential Entry Points

To effectively test for **Cross-Site Scripting (XSS)** vulnerabilities, the first step is to **map out the application** and **identify all input vectors** where user input is processed. These can include:

* **GET parameters** (e.g., `https://example.com/search?q=<payload>`).
* **POST request bodies** (e.g., comment submissions, login forms).
* **Headers** (e.g., `User-Agent`, `Referer`, `X-Forwarded-For`).
* **WebSockets** and **JSON-based APIs**.
* **Third-party integrations** (e.g., embedded iframes, content importers).

This step involves **spidering** the web application using tools like **Burp Suite**, **ZAP**, or **manual inspection** to enumerate **every parameter that accepts input**.

***

### 2. Generate Test Inputs

#### Reflection Testing

To detect reflection of input, inject **unique values** such as:

```
XSS123
<test>
'alert'
```

These values help **trace input reflection** across various parts of the application, including:

* **HTML page body**
* **Attributes**
* **JavaScript variables**
* **URL parameters**
* **Request/Response Headers**

If a value appears **unchanged in the response**, it may indicate a reflection point, which is a **potential XSS vector**.

***

### 3. Submit and Observe

#### Immediate vs. Stored Reflections

* **Reflected XSS** occurs when the injected payload is reflected **immediately** in the response.
* **Stored XSS** occurs when the payload is **saved in a database** and later executed when the stored data is retrieved.
* **DOM-Based XSS** occurs when **JavaScript on the client-side** handles and modifies the DOM using unsanitized user input.

To detect XSS, observe **browser behavior**, **page source code**, and **network responses** using **Burp Suite**, **DevTools**, and **JavaScript debugging tools**.

***

### 4. Context Analysis

Understanding the **injection context** is crucial because different contexts require **different payloads**:

#### Injection Contexts:

1.  **HTML Content**

    * If input is reflected inside HTML body:

    ```
    <div>USER INPUT</div>
    ```

    Payload:

    ```
    <script>alert(1)</script>
    ```
2.  **HTML Attributes**

    * If input is inside an HTML attribute:

    ```
    <img src="USER INPUT">
    ```

    Payload:

    ```
    " onerror="alert(1)
    ```
3.  **JavaScript Context**

    * If input is inside a JavaScript variable:

    ```
    var user = 'USER INPUT';
    ```

    Payload:

    ```
    '; alert(1); //
    ```
4.  **DOM Manipulation**

    * If JavaScript dynamically updates the page using `document.write()`, `innerHTML`, or `eval()`:

    ```
    document.write('<p>' + USER_INPUT + '</p>');
    ```

    Payload:

    ```
    </p><script>alert(1)</script>
    ```

Understanding **where** and **how** the application reflects input allows **precise payload crafting**.

***

### 5. Crafting XSS Payloads

After identifying the **injection context**, the next step is to **craft payloads that fit the context**. Consider:

* **Basic XSS Payloads**
  * `<script>alert(1)</script>`
  * `javascript:alert(1)`
* **Event Handler-Based**
  * `<img src=x onerror=alert(1)>`
  * `<svg onload=alert(1)>`
* **Obfuscation Techniques**
  * `<scr<script>ipt>alert(1)</scr<script>ipt>`
  * `<svg><a href="j&Tab;avascript:alert(1)">X</a></svg>`
* **Bypassing CSP**
  *   Using `nonce` or `hash`-based payloads:

      ```
      <script nonce="randomvalue">alert(1)</script>
      ```

Each **injection context** requires a **different payload structure**.

***

### 6. Payload Testing

Once payloads are crafted, systematically **test each input field** for execution:

* **Reflected XSS** → Verify **immediate execution** in responses.
* **Stored XSS** → Check **subsequent page loads**.
* **DOM-Based XSS** → Analyze **client-side JavaScript manipulation**.

Use tools like:

* **Burp Suite Repeater** – for manual request testing.
* **XSS Hunter** – for automated stored XSS detection.
* **DOM Invader (Burp)** – for tracing JavaScript execution paths.

***

### 7. Browser Execution and Debugging

Manually verify XSS execution using **browser-based debugging tools**:

1. **DevTools Console** – Check **JavaScript errors** and **event handlers**.
2. **DOM Manipulation Testing** – Modify elements dynamically in the browser.
3. **CSP Policies** – Use `Content-Security-Policy-Report-Only` headers to detect blocked payloads.

If a payload does **not execute**, inspect:

* **How JavaScript escapes input.**
* **Character encoding (HTML entities, URL encoding).**
* **Event handlers that may execute scripts.**

***

### 8. Documenting Payload Execution

For **each successful XSS execution**, document:

* **Payload Used**
* **Vulnerable Parameter**
* **Execution Context**
* **Impact (Session Theft, Keylogging, Phishing)**

Use screenshots and **PoC (Proof of Concept) code** for reporting.

***

### 9. Refining the Attack

If an application blocks basic payloads, attempt **bypass techniques**:

* **Encoding Variations**
  * `<img src=x onerror=&#x61;&#x6c;&#x65;&#x72;&#x74;(1)>`
* **Using Alternative Tags**
  * `<math><mtext><a xlink:href="javascript:alert(1)">Click</a></mtext></math>`
* **Breaking Out of Attributes**
  * `"><svg onload=alert(1)>`

Testing **multiple bypass methods** ensures **comprehensive security validation**.

***

### 10. Automated XSS Scanning

While **manual testing is essential**, automation helps detect **widespread XSS vulnerabilities**:

* **Burp Suite Scanner** – Detects **reflected and stored XSS**.
* **OWASP ZAP** – Open-source XSS scanning tool.
* **XSStrike** – Intelligent XSS detection framework.

Automated tools help **speed up testing**, but **manual confirmation is always required**.

***

### 11. Cross-Browser Testing

Different browsers handle JavaScript **slightly differently**. **Test XSS payloads in multiple browsers**, including:

* **Chrome**
* **Firefox**
* **Edge**
* **Safari**

Some **older browser versions** allow **more relaxed execution rules**, making them **useful for legacy testing**.

***

### 12. Testing for Stored XSS

For **Stored XSS**, check:

* **Does the payload persist across sessions?**
* **Is the payload accessible by multiple users?**
* **Can it be used for phishing, account takeover, or other high-impact exploits?**

***

### 13. Mitigation and Remediation

#### **1. Input Validation and Sanitization**

* Use **whitelist-based validation** for all user input.
* Remove or escape special characters (`<`, `>`, `"`, `'`, `&`).

#### **2. Output Encoding**

* Use **context-aware escaping** (e.g., `htmlspecialchars()` in PHP, `encodeForHTML()` in Java).

#### **3. Content Security Policy (CSP)**

*   Implement **strict CSP headers**:

    ```
    Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-randomvalue'
    ```

#### **4. HTTPOnly and Secure Cookies**

*   Prevent JavaScript from accessing session cookies:

    ```
    Set-Cookie: sessionid=xyz; HttpOnly; Secure
    ```

#### **5. JavaScript Security Controls**

* Use **Trusted Types** API to enforce **safe DOM modifications**.
* Implement **Subresource Integrity (SRI)** to verify **external script integrity**.

***

### Conclusion

A structured **XSS testing methodology** ensures **thorough detection** and **exploitation of vulnerabilities**. Following **secure coding practices**, **CSP enforcement**, and **input validation** significantly reduces the risk of XSS attacks.
