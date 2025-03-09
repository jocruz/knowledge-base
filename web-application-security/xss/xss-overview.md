# XSS Overview

## Introduction to Cross-Site Scripting (XSS)

### **High-Level Summary**

Cross-Site Scripting (**XSS**) is a **web security vulnerability** that allows attackers to inject **malicious JavaScript** into a web application, which then executes in the victim's browser. Despite modern security measures, **XSS remains one of the most prevalent web vulnerabilities**, frequently appearing in **bug bounties, penetration tests, and real-world breaches**.

#### **Why is XSS Dangerous?**

* **Data Theft:** Attackers can steal **session tokens**, **cookies**, and sensitive user data.
* **Keylogging:** Malicious scripts can record **keystrokes**.
* **Credential Theft:** Autofill credentials can be exfiltrated.
* **Defacement & Exploitation:** Attackers can modify web page content, redirect users, or inject harmful ads.
* **Malware Delivery:** XSS can be leveraged to serve **malicious payloads** to users.

### **Definition Bank**

| Term                           | Definition                                                                                              |
| ------------------------------ | ------------------------------------------------------------------------------------------------------- |
| **Cross-Site Scripting (XSS)** | A web vulnerability where an attacker injects and executes JavaScript in a user's browser.              |
| **Reflected XSS**              | The malicious script is included in a request and reflected back in the response.                       |
| **Stored XSS**                 | The payload is permanently stored (e.g., in a database) and executes for every user who loads the page. |
| **DOM-Based XSS**              | The payload is executed within the **browser DOM** without interacting with the server.                 |
| **Payload**                    | The malicious code injected by an attacker.                                                             |
| **Execution Context**          | The location where the XSS payload executes (HTML, attributes, JavaScript, etc.).                       |

### **Types of XSS Attacks**

#### **1. Reflected XSS**

* The malicious payload is included in a request and reflected in the response.
*   **Example Attack Vector:**

    ```html
    <a href="https://example.com/search?q=<script>alert('XSS')</script>">Click me</a>
    ```
* **Scope:** Affects users who click on the crafted link.

#### **2. Stored XSS**

* The attacker stores the malicious script in the database or another persistent storage.
*   **Example Attack Vector:**

    ```html
    <script>alert('Stored XSS');</script>
    ```
* **Scope:** Affects all users who load the infected page.

#### **3. DOM-Based XSS**

* The vulnerability is present **in the client-side JavaScript code**.
*   **Example Attack Vector:**

    ```js
    var userInput = location.hash.substring(1);
    document.getElementById("output").innerHTML = userInput;
    ```
* **Scope:** No interaction with the server; executes purely in the browser.

### **Common Pitfalls That Lead to XSS**

1.  **Using `innerHTML` to insert user input**

    ```js
    document.getElementById("output").innerHTML = userInput; // ❌ Vulnerable
    ```
2.  **Using `eval()` to execute user-provided input**

    ```js
    eval(userInput); // ❌ Dangerous
    ```
3.  **Not encoding special characters before rendering**

    ```html
    <div>Welcome, <span id="username">user</span></div>
    ```

    **If `username` is unescaped input, it allows script injection.**

### **Best Practices for Preventing XSS**

#### **1. Input Validation & Sanitization**

* **Sanitize** user input with libraries like **DOMPurify**.
* **Use allowlists**, rejecting unexpected characters.

#### **2. Output Encoding**

* Encode special characters before displaying user input in **HTML, JavaScript, or URLs**.
*   Use functions like:

    ```js
    encodeURIComponent(userInput);
    ```

#### **3. Content Security Policy (CSP)**

**What is CSP?** A **Content Security Policy (CSP)** restricts which scripts can be executed on a page.

**Example CSP Header:**

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com;
```

**Effect:**

* Blocks inline scripts and **untrusted third-party sources**.
* **Reduces the impact of XSS vulnerabilities**.

#### **4. Subresource Integrity (SRI)**

* Prevents loading **tampered** third-party scripts.
*   Example:

    ```html
    <script src="https://cdn.example.com/script.js" integrity="sha384-BASE64HASH" crossorigin="anonymous"></script>
    ```

#### **5. Trusted Types**

* **What it does:** Prevents insecure DOM manipulations.
*   **Example Policy:**

    ```js
    TrustedTypes.createPolicy("default", {
      createHTML: (input) => input.replace(/</g, "&lt;")
    });
    ```

#### **6. Avoid Dangerous Functions**

*   **Do not use:**

    ```js
    document.write(); // ❌
    innerHTML = userInput; // ❌
    setTimeout(userInput, 1000); // ❌
    ```
*   **Use safe alternatives:**

    ```js
    element.textContent = userInput; // ✅ Safe
    ```

### **Real-World XSS Exploits**

#### **1. MySpace Worm (Samy XSS, 2005)**

* **What happened?**
  * A **self-replicating XSS worm** was injected into MySpace profiles.
  * The worm spread automatically whenever someone viewed an infected profile.
* **Impact:** Over **1 million users** were affected in **24 hours**.
* **Mitigation:** MySpace implemented **proper input sanitization** and **CSP headers**.

#### **2. British Airways XSS Breach (2018)**

* **What happened?**
  * A compromised **third-party script** led to **card-skimming malware**.
* **Impact:** **380,000 payment details stolen**.
* **Mitigation:** Implementation of **Subresource Integrity (SRI)** and **CSP to restrict third-party scripts**.

### **Conclusion**

XSS remains a **critical web vulnerability**, with real-world cases highlighting its severity. **Implementing a defense-in-depth strategy** with **input validation, CSP, Trusted Types, and SRI** is essential to mitigating these attacks.

### **References**

* [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
* [Mozilla Developer Network - Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
* [Google Security Blog - Trusted Types](https://security.googleblog.com/2020/03/restricting-dynamic-code-evaluation.html)
