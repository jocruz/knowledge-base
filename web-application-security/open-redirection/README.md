# Open Redirection

## **Open Redirects Overview**

***

### **High-Level Summary**

**Open Redirects** occur when a web application **forwards users** to a **URL** provided via **user input** without proper validation. Attackers exploit this vulnerability to redirect users to **malicious websites**, facilitating **phishing attacks, malware distribution, and credential theft**.

***

### **Definition Bank**

| **Term**                    | **Definition**                                                                                         |
| --------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Open Redirect**           | A vulnerability where an application allows user-controlled input to determine a redirect destination. |
| **Query Parameter**         | A part of a URL that passes data (`?redirectURL=dashboard`).                                           |
| **DOM-Based Vulnerability** | A client-side vulnerability where the **DOM** processes untrusted inputs.                              |
| **Phishing Attack**         | Deception technique used to trick users into revealing credentials via fake websites.                  |

***

### **How Open Redirects Work (Step-by-Step)**

1. **User Login:**
   * The user successfully logs into a legitimate website.
2. **Legitimate Redirect Mechanism:**
   *   The application redirects the user via a **query parameter** after login:

       ```
       https://secure-website.com/login?redirectURL=https://secure-website.com/dashboard
       ```
3. **Exploiting the Redirect:**
   *   An attacker modifies the **redirectURL** parameter to point to their malicious site:

       ```
       https://secure-website.com/login?redirectURL=https://malicious.com
       ```
4. **User Clicks the Link:**
   * The victim **trusts** the secure-website.com domain and clicks the link.
5. **Redirection to a Malicious Website:**
   * The user is redirected to `https://malicious.com`, where attackers may:
     * **Steal credentials** by mimicking the login page.
     * **Deploy malware** via drive-by downloads.
     * **Harvest session cookies** using social engineering techniques.

***

### **Real-World Incidents & CVEs Related to Open Redirects**

#### **CVE-2014-3783: PayPal Open Redirect Vulnerability**

* **Issue:** PayPal’s website contained an **open redirect** that allowed attackers to redirect users to phishing pages.
* **Exploitation:** Attackers crafted PayPal-branded URLs to trick users into entering login credentials on **malicious phishing sites**.
* **Fix:** PayPal implemented a **redirect allowlist** to restrict URLs.
* **Reference:** [CVE-2014-3783 - PayPal Open Redirect](https://nvd.nist.gov/vuln/detail/CVE-2014-3783)

***

#### **CVE-2019-17520: Apache HTTP Server Open Redirect**

* **Issue:** The Apache HTTP server had a vulnerability where certain redirects could be manipulated by user input.
* **Exploitation:** Attackers could craft URLs that redirected users to malicious destinations.
* **Fix:** Updated server-side handling to prevent **user-controlled redirects**.
* **Reference:** [CVE-2019-17520 - Apache Open Redirect](https://nvd.nist.gov/vuln/detail/CVE-2019-17520)

***

#### **Example Exploit Scenario**

**Original URL (Legitimate Redirect)**

```
https://secure-website.com/login?redirectURL=https://secure-website.com/dashboard
```

**Malicious URL (Exploiting Open Redirect)**

```
https://secure-website.com/login?redirectURL=https://malicious.com
```

**Why This Works**

* The website **fails to validate** the destination URL.
* The attacker **abuses user trust** by making the URL appear to belong to `secure-website.com`.

***

### **How to Identify Open Redirect Vulnerabilities**

#### **1. Manual Testing**

* Inspect **redirect parameters** such as:
  * `?redirect=`
  * `?next=`
  * `?returnTo=`
* Modify them to external domains and check if redirection occurs.

#### **2. Automated Scanning**

* Use tools like **Burp Suite Active Scanner** or **OWASP ZAP** to detect open redirect vulnerabilities.

#### **3. DOM-Based Open Redirect Analysis**

*   Look for **JavaScript handling of redirects**:

    ```javascript
    window.location.href = new URLSearchParams(window.location.search).get("redirectURL");
    ```
* Check if it allows **unvalidated external URLs**.

***

### **Preventing Open Redirect Vulnerabilities**

#### **1. Implement Redirect Allowlisting**

* Allow redirections **only** to predefined trusted domains.

```python
ALLOWED_REDIRECTS = [
    "https://secure-website.com/dashboard",
    "https://secure-website.com/profile"
]

def secure_redirect(url):
    if url in ALLOWED_REDIRECTS:
        return redirect(url)
    else:
        return "Invalid redirect URL", 400
```

#### **2. Use Relative URLs**

* Instead of allowing **full external URLs**, restrict redirects to internal paths.

```javascript
// Secure approach: Allow only internal redirections
const redirectURL = new URL(window.location.origin + new URLSearchParams(window.location.search).get("redirectURL"));
if (redirectURL.origin === window.location.origin) {
    window.location.href = redirectURL;
} else {
    alert("Invalid redirect");
}
```

#### **3. Validate User Input**

* Use **server-side validation** to ensure URLs match expected patterns.

```php
$redirectURL = $_GET['redirectURL'];

if (!preg_match('/^\/[a-zA-Z0-9\/]+$/', $redirectURL)) {
    die("Invalid redirect URL");
}

header("Location: " . $redirectURL);
```

#### **4. Encode Output Properly**

* If redirection parameters are embedded in HTML, apply **output encoding**.

```html
<a href="/redirect?next=<?php echo urlencode($nextPage); ?>">Continue</a>
```

***

### **Security Implications of Open Redirects**

#### **1. Phishing Attacks**

* Attackers craft links impersonating **legitimate websites** but redirect users to **malicious login pages**.

#### **2. Malware Distribution**

* Redirects lead to **exploit kits** or **drive-by downloads**.

#### **3. Session Hijacking**

* Attackers use open redirects to steal **session cookies** via cross-site scripting (XSS) techniques.

***

### **Mitigation Summary**

| **Best Practice**                | **Implementation**                                            |
| -------------------------------- | ------------------------------------------------------------- |
| **Use Allowlisting**             | Only allow predefined redirect destinations.                  |
| **Restrict to Relative URLs**    | Avoid absolute URLs, only allow internal redirects.           |
| **Validate Redirect Parameters** | Ensure the redirect destination is within an expected domain. |
| **Encode Output Properly**       | Prevent XSS exploitation of redirects.                        |

***

### **Key Takeaways**

* **Open Redirects are widely exploited** in phishing and malware campaigns.
* **Query parameters like `redirectURL`, `next`, or `returnTo`** are common sources of this vulnerability.
* **Mitigations include allowlisting, relative paths, and strict validation**.
* **Security measures must be enforced both on the client and server sides**.

***

### **Further Reading & References**

* **OWASP Open Redirect Cheat Sheet**: [OWASP Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Open_Redirect_Cheat_Sheet.html)
* **CVE-2014-3783 (PayPal Open Redirect Issue)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2014-3783)
* **CVE-2019-17520 (Apache Open Redirect Vulnerability)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2019-17520)
