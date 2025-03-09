# Reflected XSS in JavaScript String with Angle Brackets Encoded (PortSwigger Lab)

## Lab: Reflected XSS in JavaScript String with Angle Brackets Encoded

### Lab URL

[https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded)

### High-Level Summary

**Objective:** Perform a reflected XSS attack that breaks out of a JavaScript string and calls the `alert` function.

#### Key Techniques & Concepts

* Reflected Cross-Site Scripting (XSS)
* JavaScript String Injection
* Breaking Out of JavaScript Context

#### What Makes This Vulnerable?

* The web application reflects user input directly into a JavaScript string without proper encoding.
* Angle brackets (`< >`) are encoded, but it fails to properly escape single quotes or string-breaking payloads.

***

### Steps to Exploit

1. **Initial Test:**
   * Submit a random alphanumeric string in the search box.
   * Use **Burp Suite** to intercept the request and send it to **Burp Repeater**.
2. **Reflection Check:**
   * Observe the random string reflected inside a JavaScript string in the response.
3. **Exploit Attempt:**
   *   Replace the random string with:

       ```
       '-alert(1)-'
       ```
   * Send the modified request and verify the payload is reflected in the response.
4. **Execution Verification:**
   * Reload the page to trigger the `alert()` function.

***

### Explanation of Payloads and Techniques

#### Why `'-alert(1)-'` Works

* The payload includes a single quote (`'`) to break out of the JavaScript string.
* `-alert(1)-` is inserted into the JavaScript context as executable code.
* The hyphens (`-`) are ignored as comments since they do not affect JavaScript execution in this context.

#### Why `qwerty';prompt()//';` Works

* The single quote (`'`) breaks out of the JavaScript string.
* `prompt()` executes a JavaScript dialog box.
* The double forward slashes (`//`) comment out the remaining code to avoid syntax errors.

#### Why `"><script>alert(1)</script>` Fails

* Since angle brackets are encoded, the `<script>` tag is rendered ineffective.
* Breaking out of a JavaScript string requires bypassing the quote escape mechanism.

#### Key Payload Components

* `'-alert(1)-'`: Breaks out of the string and executes code.
* `qwerty';prompt()//';`: Achieves the same by commenting out the remaining code.
* **Commenting the Line (`//`)** prevents the remaining code from breaking.

***

### Tools and Commands

* **Burp Suite:** Repeater, Intruder
* **Payload:** `'-alert(1)-'`
* **Resources:** [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
