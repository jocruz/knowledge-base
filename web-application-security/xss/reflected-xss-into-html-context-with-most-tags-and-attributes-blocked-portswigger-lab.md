# Reflected XSS into HTML Context with Most Tags and Attributes Blocked (PortSwigger Lab)

## Lab: Reflected XSS into HTML Context with Most Tags and Attributes Blocked

### **Lab URL**

[https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)

### **High-Level Summary**

#### **Objective**

Perform a **Reflected XSS** attack that bypasses a web application firewall (**WAF**) and successfully executes the `print()` function in the victim’s browser.

#### **Key Techniques & Concepts**

* **Reflected XSS** – Injecting a script that executes within the victim’s browser via a reflected input.
* **Web Application Firewall (WAF) Bypass** – Identifying restrictions and crafting a payload to evade filtering.
* **Burp Suite Intruder** – Automating payload discovery to find allowed tags and event handlers.

#### **Vulnerability Factors**

* The web application **reflects user input** into the HTML response without proper sanitization.
* A **WAF is in place**, but it does not comprehensively filter all XSS payloads.

***

### **PortSwigger Solution**

#### **1. Initial Test**

Inject a basic payload to check for execution:

```
<img src=1 onerror=print()>
```

* **Observation:** This payload is blocked by the WAF.

#### **2. Identifying Allowed Tags**

* Navigate to the search bar and enter `qwerty`.
* Intercept the request in **Burp Suite** and send it to **Intruder**.
* Replace `qwerty` with `<§§>` and set a payload position marker.
* Use the [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) to import a list of **HTML tags**.
* **Run the Intruder attack** and sort responses by status codes.
* **Discovery:** The `<body>` tag returns a **200 status code**, indicating it is allowed.

#### **3. Identifying Allowed Event Handlers**

* Modify the payload:

```
<body%20§§=1>
```

* Use **Burp Intruder** with an **event handler list** from the XSS Cheat Sheet.
* **Discovery:** `onresize` is allowed and returns a **200 status code**.

#### **4. Crafting the Final Payload**

Construct a payload that forces execution upon window resize:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

* Host the payload on **Burp Suite’s Exploit Server**.
* Send the payload link to the victim.
* Upon page load, the iframe resizes and triggers the **onresize** event, executing the `print()` function.

***

### **Instructor Solution**

#### **1. Identifying the Reflection Point**

* Enter `qwerty` into the search field and view the **source code**.
* Confirm that the input is **reflected within `<h1>` tags**.

#### **2. Testing Initial Payloads**

* Inject `<script>prompt()</script>` – blocked.
* Inject `<img src=x onerror=prompt()>` – blocked.

#### **3. Discovering Allowed Tags**

* Send the request to **Burp Repeater**.
* Use **Burp Intruder** with HTML tags from the **XSS Cheat Sheet**.
* Observe that `<body>` is allowed.

#### **4. Testing Event Handlers**

* Modify `<body>` with different event handlers.
* **Finding:** `onresize` executes JavaScript and is not blocked.

#### **5. Encoding Considerations**

* **Observation:** Special characters undergo **automatic URL encoding**.
* Adjust payload encoding as necessary.

#### **6. Final Payload for Execution**

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='1em'>
```

* **Delivery Method:** Host the payload on an **exploit server** and send the malicious link to the victim.

***

### **Explanation of Techniques Used**

#### **Why This Payload Works**

* The **WAF blocks** `<script>` and `<img>` tags.
* The `<body>` tag is allowed, enabling **event-based execution**.
* The `onresize` event executes when the **iframe resizes**.

#### **Key Components of the Payload**

| Component  | Purpose                                     |
| ---------- | ------------------------------------------- |
| `<iframe>` | Embeds another web page                     |
| `onresize` | Executes JavaScript when the window resizes |
| `print()`  | Triggers a print dialog, proving execution  |

***

### **XSS Attack Workflow**

1. **Identify Reflection Points** – Find where user input is reflected.
2. **Test Initial Payloads** – Use `<script>alert(1)</script>` and `<img onerror=alert(1)>`.
3. **Enumerate Allowed Tags** – Use Burp Intruder to test for allowed tags.
4. **Test Event Handlers** – Find event attributes that execute JavaScript.
5. **Check Encoding Behaviors** – Verify how special characters are processed.
6. **Craft the Final Payload** – Use an event-driven method for execution.
7. **Exploit Delivery** – Deliver the payload via an exploit server.

***

### **Best Practices for Preventing XSS**

* **Input Validation:** Sanitize user input to block XSS attempts.
* **Output Encoding:** Ensure special characters are escaped.
* **Content Security Policy (CSP):** Restrict script execution sources.
* **Avoid Dangerous Functions:** Do not use `innerHTML`, `eval()`, `document.write()`.
* **Use Trusted Types:** Enforce secure JavaScript handling.

***

### **Conclusion**

This lab demonstrates **how to bypass a WAF** using a **Reflected XSS attack**. By understanding **tag allowances, event handlers, and encoding behaviors**, attackers can craft payloads that evade filtering. Secure applications must employ **strict input validation, CSP policies, and encoding techniques** to mitigate such vulnerabilities.
