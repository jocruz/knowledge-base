# Dom-Based Open Redirection (PortSwigger Lab)

## **Lab: DOM-Based Open Redirection**

**Lab URL:** [Web Security Academy](https://portswigger.net/web-security/dom-based/open-redirection/lab-dom-open-redirection)

***

### **High-Level Summary**

#### **Objective**

Exploit a DOM-based open redirection vulnerability by manipulating the JavaScript logic responsible for handling URL redirection. This will redirect the victim to an external exploit server.

#### **Key Techniques & Concepts**

* **DOM Manipulation:** The vulnerability stems from unvalidated user input being processed in JavaScript.
* **Open Redirection:** The `location.href` property is used improperly, allowing external redirects without validation.
* **Client-Side Execution:** Since the vulnerability is in JavaScript, the attack occurs within the victim’s browser rather than on the server.

#### **Why This is Vulnerable**

* The application extracts and processes the `url=` parameter without sanitization.
* JavaScript on the client-side executes the redirection without verifying the target domain.
* An attacker can craft a malicious link that redirects users to a phishing site or exploit server.

***

### **PortSwigger Solution**

#### **Step 1: Identify the Vulnerability**

* Navigate to a blog post within the web application.
* Locate and inspect the **Back to Blog** button.
* Observe the following vulnerable JavaScript snippet in the `onclick` event:

```html
<a href="#" onclick="returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : '/';">Back to Blog</a>
```

#### **Step 2: Analyze the Vulnerable JavaScript**

* The function checks if the current page URL contains `url=`.
* If found, it assigns the extracted URL to `location.href`, triggering an automatic redirection.
* If not found, it defaults to redirecting the user to `/`.

#### **Step 3: Craft the Malicious URL**

To exploit this behavior, construct a URL that includes an external redirection:

```
https://YOUR-LAB-ID.web-security-academy.net/post?postId=4&url=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/
```

#### **Step 4: Execute the Exploit**

* Paste the crafted URL into the browser’s address bar and press **Enter**.
* If successful, the JavaScript extracts the `url` parameter and redirects to the **exploit server**.
* The redirection confirms the vulnerability, completing the lab.

***

### **Instructor Solution**

#### **Step 1: Inspect the Web Page**

* Visit the vulnerable web application.
* Navigate to a blog post and examine the **Back to Blog** button.
* Open the browser’s developer tools and review the associated JavaScript.

#### **Step 2: Understand the Vulnerable Code**

The JavaScript processes the URL using the following regex:

```javascript
/url=(https?:\/\/.+)/
```

**Regex Breakdown:**

| Component | Description                                                                       |
| --------- | --------------------------------------------------------------------------------- |
| `url=`    | Looks for the `url=` parameter in the URL.                                        |
| `https?`  | Matches either `http` or `https`.                                                 |
| `:\/\/`   | Matches `://` to form a valid URL structure.                                      |
| `.+`      | Matches the remainder of the URL, including domains, paths, and query parameters. |

#### **Step 3: Craft the Exploit**

*   Construct a URL with a malicious redirection target:

    ```
    https://YOUR-LAB-ID.web-security-academy.net/post?postId=4&url=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/
    ```
* Copy this URL and paste it into the browser.

#### **Step 4: Verify the Exploit**

* Upon pressing **Enter**, the JavaScript executes the redirection.
* The browser navigates to the **exploit server**, confirming successful exploitation.

***

### **Security Implications**

#### **Potential Real-World Risks**

* **Phishing Attacks:** Attackers can redirect victims to a fake login page, capturing credentials.
* **Session Hijacking:** If used with a social engineering attack, victims may unknowingly expose session tokens.
* **Data Exfiltration:** Open redirections can leak sensitive parameters, such as authentication tokens or user identifiers.

#### **Real-World Examples**

* **CVE-2021-44746:** Google OAuth vulnerability that allowed open redirections, leading to potential token theft.\
  [Reference: CVE-2021-44746](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44746)
* **Facebook OAuth Flaw (2019):** Attackers exploited open redirections in Facebook’s OAuth system to steal access tokens.\
  [Reference: Facebook Security Issue](https://www.securityweek.com/researcher-wins-30000-facebook-security-bug)

***

### **Mitigation Strategies**

#### **1. Implement Allowlisting for Redirects**

A secure way to handle redirects is by restricting redirection targets to trusted domains:

```javascript
const allowedDomains = ['example.com'];
const params = new URLSearchParams(window.location.search);
const returnUrl = params.get('url');

if (returnUrl && allowedDomains.some(domain => returnUrl.startsWith(domain))) {
    window.location.href = returnUrl;
} else {
    window.location.href = '/';
}
```

#### **2. Enforce Server-Side Validation**

Ensure that any redirects are validated before execution:

```python
# Flask Example
from flask import request, redirect

ALLOWED_DOMAINS = ['example.com']

def safe_redirect(target_url):
    if target_url.startswith(tuple(ALLOWED_DOMAINS)):
        return redirect(target_url)
    return redirect('/')
```

#### **3. Avoid Client-Side URL Handling**

Instead of processing redirections on the client, enforce server-side logic for controlled redirects.

#### **4. Use Relative Paths Instead of Full URLs**

If redirection is necessary, limit it to internal paths:

```javascript
const safePath = returnUrl.startsWith('/') ? returnUrl : '/';
window.location.href = safePath;
```

***

### **Conclusion**

* **DOM-based open redirections occur due to unvalidated URL parameters being processed in JavaScript.**
* **Attackers can exploit this flaw to redirect users to malicious sites, leading to phishing and session hijacking.**
* **Proper validation, domain allowlisting, and removal of open redirect functions are necessary to mitigate this risk.**
