# Stored XSS into Anchor href Attribute with Double Quotes HTML-Encoded (PortSwigger Lab)

## Lab: Stored XSS into Anchor `href` Attribute with Double Quotes HTML-Encoded

### Lab URL

[https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded)

### High-Level Summary

**Objective:** Exploit a stored XSS vulnerability by submitting a comment that calls the `alert` function when the comment author's name is clicked.

#### Key Techniques & Concepts

* Stored Cross-Site Scripting (XSS)
* HTML Attribute Injection
* URL Scheme Injection

#### What Makes This Vulnerable?

* The web application reflects user input inside an anchor `href` attribute without proper validation.
* Double quotes are HTML-encoded, but other payload techniques can bypass this restriction.

***

### Steps to Exploit

1. **Initial Test:**
   * Post a comment with a random alphanumeric string in the "Website" input.
   * Use **Burp Suite** to intercept the request and send it to **Burp Repeater**.
2. **Reflection Check:**
   * Make a second request to view the post and intercept it with Burp Suite.
   * Confirm the random string appears inside an anchor (`href`) attribute in the source code.
3. **Exploit Attempt:**
   *   Repeat the test, but replace the random string with:

       ```
       javascript:alert(1)
       ```
   * Send the modified request and verify the payload is stored.
4. **Execution Verification:**
   * Right-click the comment link, select **Copy URL**, and paste it into the browser.
   * Clicking the comment author's name should trigger the `alert()` box.

***

### Explanation of Payloads and Techniques

#### Why `javascript:alert(1)` Works

* The `javascript:` URL scheme executes JavaScript directly when the anchor link is clicked.
* Since double quotes are HTML-encoded, injecting directly into the `href` attribute works because the payload does not require quotes.

#### Why `"><script>alert(1)</script>` Fails

* Double quotes are encoded, and the `<script>` tag gets sanitized or escaped.
* The browser prevents inline script execution due to encoding and modern security controls.

#### Key Payload Components

* `javascript:`: Scheme to execute inline JavaScript.
* `alert(1)`: Simple JavaScript to demonstrate code execution.

***

### What is `javascript:` and Why It Works

The **URI scheme** (Uniform Resource Identifier scheme) defines how the browser interprets a link. The `javascript:` scheme allows code execution directly when the link is clicked without breaking out of the `href` attribute context. **Browsers support multiple URI schemes**, including `https:`, `mailto:`, `ftp:`, and `javascript:`.

#### How It Works

* The `javascript:` scheme directly runs JavaScript code inside the `href` attribute.
* It does not require breaking out of quotes since it fits naturally within the attribute.

#### Example

```
<a href="javascript:alert(1)">Click me!</a>
```

#### Why Other Payloads Fail

* `<script>` tags often fail because modern browsers encode double quotes and prevent inline script execution.

**Key Insight:** The `javascript:` scheme works because it allows JavaScript execution directly inside the `href` without breaking the HTML structure.

***

### Tools and Commands

* **Burp Suite:** Repeater, Intruder
* **Payload:** `javascript:alert(1)`
* **Resources:** [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
