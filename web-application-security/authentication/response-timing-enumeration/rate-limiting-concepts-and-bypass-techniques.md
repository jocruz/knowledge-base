---
description: >-
  This section explores rate limiting, how it prevents abuse, and common
  techniques used to bypass it.
---

# Rate Limiting (Concepts & Bypass Techniques)

## **What is Rate Limiting?**

Rate limiting is a security control used to **restrict the number of requests** a user can send within a given timeframe. It helps prevent:

* **Brute-force attacks** by limiting login attempts.
* **Denial of Service (DoS) attacks** by reducing request floods.
* **Web scraping and API abuse** by throttling repeated requests.

#### **How It Works**

1. A login form or API **tracks each request** by logging the user’s **IP address, session token, or cookie**.
2. A **counter** increases after each request.
3. If more than a defined number of attempts are made within a set timeframe, the request is **blocked or delayed**.

#### **Example Implementation**

* **10 login attempts per minute** → Temporary block for 5 minutes.
* **100 API requests per second** → IP throttled.
* **3 failed payment attempts** → Captcha required.

***

### **Checklist for Testing Rate Limiting**

#### **1. Identify the Rate Limiting Mechanism**

* Is the limit based on **IP, cookies, or session tokens**?
* What triggers the block? (Login attempts, requests per second?)
* How long does the block last?

#### **2. Test Rate Limit Bypass Techniques**

| **Bypass Method**          | **Description**                                        |
| -------------------------- | ------------------------------------------------------ |
| **IP Spoofing**            | Modify `X-Forwarded-For`, `X-Real-IP`, or `Client-IP`. |
| **Session Token Rotation** | Reset cookies/session tokens after each request.       |
| **User-Agent Rotation**    | Change browser/device identifiers.                     |
| **Tampering HTTP Methods** | Use alternate verbs (e.g., `POST` vs. `GET`).          |
| **Slow Brute Force**       | Reduce request rate to avoid detection.                |
| **Distributed Attacks**    | Send requests from multiple IPs via proxies/VPNs.      |

#### **3. Observe Server Behavior**

* Does **changing headers or cookies** reset the counter?
* Does **using different HTTP methods** affect rate limits?
* Does the **ban persist across sessions/IPs?**

***

### **Bypassing Rate Limits: Common Techniques**

#### **1. IP Address Spoofing (Header Manipulation)**

Many applications rely on **IP-based rate limiting**, which can often be bypassed by modifying request headers.

**Modify the following headers to spoof a new IP:**

```plaintext
X-Real-IP: 192.168.1.100
X-Forwarded-For: 192.168.1.100
Client-IP: 192.168.1.100
True-Client-IP: 192.168.1.100
```

If modifying these headers **resets the rate limit**, the application **relies on client-provided headers**, making it vulnerable.

***

#### **2. Session & Cookie Manipulation**

Some applications **track login attempts via session tokens instead of IPs**.

**Test bypass by:**

* Clearing cookies before each request.
* Generating new session tokens manually.
* Testing anonymous (incognito) mode vs. logged-in state.

If **resetting cookies or logging out bypasses the limit**, the application **tracks sessions rather than IPs**.

***

#### **3. User-Agent Rotation**

Some applications track request frequency based on **browser fingerprints**.

**Modify the `User-Agent` header** to appear as different devices.

```plaintext
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 15_2 like Mac OS X)
```

If changing `User-Agent` resets the limit, then fingerprinting is being used instead of proper server-side validation.

***

#### **4. Slow Brute Force Attacks**

Some applications **only track rapid repeated requests**.

**Test bypass by:**

* Sending **one request every few seconds** (mimicking human behavior).
* Observing if **requests are still accepted despite failed attempts**.

If slowing down requests avoids triggering rate limits, the system **only detects high-frequency attacks**.

***

#### **5. Distributed Requests (Using Proxies/VPNs)**

Instead of sending **multiple requests from a single IP**, attackers use **different IP addresses**.

**Test bypass by:**

* Using a **proxy service or VPN**.
* Sending requests from **multiple cloud instances**.

If the block is **only tied to IP address**, the application **does not have proper distributed tracking**.

***

### **Headers Used in Rate Limiting**

| **Header Name**    | **Purpose**                                         |
| ------------------ | --------------------------------------------------- |
| `X-Real-IP`        | Shows the original client IP (used by proxies).     |
| `X-Forwarded-For`  | Lists all IPs the request passed through.           |
| `X-Originating-IP` | Alternative client IP tracking header.              |
| `Client-IP`        | Some servers log the client’s IP using this header. |
| `True-Client-IP`   | Used for accurate client identification.            |

If the application relies on any of these headers for rate limiting, modifying them could reset the limit.

***

### **Why Do Developers Implement Rate Limiting?**

* **Prevent Abuse** – Stops brute-force attacks & spam bots.
* **Maintain Performance** – Prevents request flooding.
* **Enhance Security** – Detects suspicious behavior.

However, **poorly configured rate limits** can be:

* **Easily bypassed** (via IP spoofing or session resets).
* **Too aggressive**, causing usability issues.

***

### **Where is Rate Limiting Implemented?**

1. **Backend (Server-Side)**
   * Middleware (e.g., `express-rate-limit`, `Django Ratelimit`).
   * Custom logic inside authentication systems.
2. **Proxies & Load Balancers**
   * Cloudflare, NGINX, AWS WAF, etc.
3. **Application Logic**
   * API authentication & login protection.

Attackers often target applications with weak or misconfigured rate-limiting rules.

***

### **Brief History of Rate-Limiting Headers**

* **X-Forwarded-For**: Introduced in the late 1990s by **Squid Proxy** to track original client IPs.
* **X-Real-IP**: Popularized by **NGINX** for simpler single-IP tracking.
* These headers **became standard for tracking user requests** but can be **manipulated if not properly validated**.

***

### **Real-World Observations: Bypassing Rate Limits**

* **Header Manipulation Worked**
  * Modifying `X-Forwarded-For` **bypassed login attempt tracking**.
  * Proves that the application **relied on untrusted client-side headers**.
* **Slow Brute-Force Attacks Were Effective**
  * Delaying login attempts **prevented detection**.
  * Indicates **poor anomaly detection mechanisms**.
* **Session Resetting Circumvented Limits**
  * Resetting cookies or logging out **removed restrictions**.
  * Suggests **rate limiting was tied to session tracking** instead of a global user policy.

***

### **Key Takeaways**

* **Identify rate-limiting mechanisms before testing bypasses.**
* **Test multiple techniques (headers, cookies, delays).**
* **If IP-based tracking is detected, spoof headers or use proxies.**
* **If session-based tracking is detected, reset session identifiers.**

***

#### **Next Steps**

Now that you've learned about **rate limiting and bypass techniques**, the next step is applying these concepts in a **hands-on lab**:

➡️ **\[\[Lab: Username Enumeration via Response Timing]]**

***
