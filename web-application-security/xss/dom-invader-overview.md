# DOM Invader Overview

## DOM Invader Overview

### What is DOM Invader?

**DOM Invader** is a **client-side security testing tool** integrated into **Burp Suite Professional**. It is designed to identify and exploit **DOM-based vulnerabilities**, primarily **DOM-based Cross-Site Scripting (XSS)**.

This tool works within **Burp's embedded Chromium browser** and helps security testers track how **untrusted user input** moves through the **Document Object Model (DOM)** of a web application.

***

### When to Use DOM Invader

DOM Invader is useful in scenarios such as:

* **Testing for DOM-based XSS** vulnerabilities in web applications.
* **Analyzing client-side JavaScript behavior** to find security flaws.
* **Tracing untrusted data flows** from input sources to execution points (sinks).
* **Manually identifying DOM-based vulnerabilities** in penetration tests.

***

### Key Components of DOM Invader

#### 1. Canary Tokens (Markers)

A **canary token** is a unique **marker injected into input fields** to track how data propagates within the DOM.

* If the canary token appears in a **sink (execution point)** without sanitization, it suggests a **potential vulnerability**.
* This technique helps testers pinpoint **dangerous data flows** in JavaScript.

**Example:**

If the token `abc123` is placed in `window.location.hash` and later appears in `innerHTML`, it could lead to **DOM XSS** if no sanitization is applied.

***

#### 2. Sinks (Execution Points)

A **sink** is where **data is processed or executed** within JavaScript. If **untrusted user input** reaches a sink **without validation or sanitization**, it may introduce **security vulnerabilities** such as XSS.

**Common Dangerous Sinks:**

| **Sink**           | **Potential Risk**                                                    |
| ------------------ | --------------------------------------------------------------------- |
| `innerHTML`        | Executes JavaScript if a script tag is injected                       |
| `document.write()` | Allows direct modification of the page, potentially executing scripts |
| `eval()`           | Executes arbitrary JavaScript, leading to code execution              |
| `setTimeout()`     | Runs JavaScript dynamically if user-controlled input is included      |

***

#### 3. Sources (Input Points)

A **source** is where **user-controllable data** enters the web application.

**Common Sources of Untrusted Data:**

| **Source**          | **Risk**                                                 |
| ------------------- | -------------------------------------------------------- |
| `window.location`   | Can be manipulated via the URL                           |
| `document.referrer` | Can be controlled by external sites                      |
| `localStorage`      | Stores persistent data that may be injected into the DOM |
| `innerText`         | Displays user-generated content on the page              |

***

#### 4. Vulnerability Detection Workflow

DOM Invader operates in **four main stages**:

1. **Input Monitoring** – Identifies untrusted user input sources.
2. **Canary Injection** – Inserts **unique markers** into inputs to track their flow.
3. **Payload Injection** – Tests for **XSS execution** within JavaScript contexts.
4. **Sink Analysis** – Highlights dangerous **execution points** where XSS may occur.

***

### How to Activate DOM Invader in Burp Suite Pro

1. Open **Burp Suite Professional**.
2. Navigate to **Proxy > Intercept > Open Browser**.
3. Click the **Burp Extension Icon** in the browser toolbar.
4. Enable **DOM Invader** from the settings menu.
5. Adjust settings for **canary injection** and **sink monitoring** based on the target application.

***

### Use Cases for DOM Invader

* **Identifying DOM-based XSS vulnerabilities** in JavaScript-heavy applications.
* **Tracking the movement of user input** through JavaScript functions and the DOM.
* **Detecting untrusted input reaching unsafe sinks** that could execute malicious scripts.
* **Enhancing manual penetration testing workflows** for client-side security assessments.

***

### Key Takeaways

* **DOM Invader simplifies client-side security testing**, making it easier to find vulnerabilities in JavaScript-heavy web applications.
* **Canary tokens help track untrusted data**, improving visibility into potential security flaws.
* **Sinks are execution points** where **unsanitized user input can lead to security risks**.
* **DOM Invader is available only in Burp Suite Professional** and should be used alongside other manual testing techniques.

***

### Best Practices for DOM-Based XSS Testing

* **Use canaries to track input flow** and determine if it reaches a sink.
* **Identify dangerous execution points (sinks)** and check for proper sanitization.
* **Analyze JavaScript functions handling user input** to detect insecure processing.
* **Combine DOM Invader results with manual JavaScript auditing** for comprehensive testing.
* **Use Content Security Policy (CSP) to mitigate XSS risks** by restricting inline script execution.

By integrating **DOM Invader into a penetration testing workflow**, security professionals can efficiently **detect and exploit DOM-based vulnerabilities**, strengthening web application security.
