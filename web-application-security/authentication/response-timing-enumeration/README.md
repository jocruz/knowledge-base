---
description: >-
  This section explores response timing attacks, which exploit variations in
  authentication response times to enumerate valid usernames and analyze
  rate-limiting mechanisms.
---

# Response Timing Enumeration

## **Response Timing Enumeration**

***

### **Concept Overview**

**Response timing attacks** leverage differences in how long a web application takes to respond to authentication requests. These differences can be used to infer:

* **Valid usernames** – Some applications process valid usernames differently, causing a delay.
* **Incorrect rate-limiting configurations** – Response time variations reveal ineffective or bypassable security controls.

By analyzing response times, attackers can gain **unauthorized access insights** without triggering traditional security alerts.

***

### **Key Concepts**

#### **1. Authentication Timing Analysis**

**How Response Timing Works**

* When credentials are submitted:
  * **Valid username + incorrect password** → The system processes the username and hashes the password → **Slight delay in response time**.
  * **Invalid username** → Immediate rejection (password is never checked) → **Faster response time**.

**Key Observations**

* Applications using slow hashing algorithms (**bcrypt, PBKDF2, Argon2**) amplify timing differences.
* **Longer passwords increase processing time**, making timing discrepancies more noticeable.
* Attackers can enumerate valid usernames by testing **response delays**.

**Practical Use Case**

* **Invalid usernames** → Instant rejection.
* **Valid usernames + long passwords** → Slower response due to password hashing.

***

### **Example Scenario**

1. **Submit login credentials**:
   * **Username:** `Jeremy`
   * **Password:** `cheesecake2024`
2. **Observe response times**:
   * **Invalid Username**: Immediate response (password check skipped).
   * **Valid Username**: Delayed response due to password hashing.

**Conclusion:** The username `Jeremy` exists in the system.

***

### **Tools & Techniques**

#### **Burp Suite**

**1. Repeater**

* **Monitor Response Time**: Check the bottom-right corner in Burp Suite for exact response times.

**2. Intruder**

* **Set up payloads to test usernames**.
* **Enable response timing analysis** by adding columns:
  * **Response Received**
  * **Response Completed**
* **Analyze response delays** to determine valid usernames.

***

### **Bypassing Rate Limits**

#### **1. Rate Limiting Basics**

* Prevents brute-force attempts by tracking **IP, cookies, or session tokens**.
* Example: **Blocking an IP after 10 failed logins in 1 minute**.

#### **2. Common Bypass Techniques**

| **Bypass Technique**           | **Method**                                                     |
| ------------------------------ | -------------------------------------------------------------- |
| **IP Address Rotation**        | Modify `X-Forwarded-For`, `X-Real-IP`, or `Client-IP` headers. |
| **Session Token Manipulation** | Reset cookies between login attempts.                          |
| **User-Agent Rotation**        | Change browser/device identifiers.                             |
| **Slow Attacks**               | Reduce request rate to avoid detection.                        |
| **Distributed Requests**       | Use multiple IPs from cloud services.                          |

***

### **Next Steps**

#### **Checklist**&#x20;

✅ **Monitor Response Timing Differences** → Validate findings with repeated tests.\
✅ **Rate Limiting Testing** → Identify **tracking mechanisms** (IP, cookies, headers).\
✅ **Troubleshooting** → Consider network latency & adjust testing methodology.

#### **Next Steps**

* Once valid usernames are identified:
  * Proceed with **password brute forcing**.
  * Explore **credential stuffing attacks**.
  * Investigate other enumeration vulnerabilities.

***

