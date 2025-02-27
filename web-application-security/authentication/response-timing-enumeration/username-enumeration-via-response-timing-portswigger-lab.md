---
description: >-
  A hands-on lab demonstrating how response timing variations can be used to
  enumerate valid usernames and analyze rate-limiting defenses.
---

# Username Enumeration via Response Timing (PortSwigger Lab)

***

### **Lab Overview**

🔍 **Objective:** Identify valid usernames using **response timing analysis** and test for **rate limit bypass techniques**.

[Lab: Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

🛠 **Techniques Used:**

* Response time-based **username enumeration**.
* **Header manipulation (`X-Forwarded-For`)** to bypass lockouts.
* **Burp Suite Intruder (Pitchfork Attack Type)** for automated analysis.

***

### **Steps to Complete the Lab**

#### **Step 1: Locate the Login Request**

1. Attempt to log in with invalid credentials.
2. In **Burp Suite**, find the **`POST /login`** request in **HTTP history**.
3. Send the request to **Repeater** for further testing.

***

#### **Step 2: Test Lockout Behavior**

1. Send multiple **failed login attempts** using the same credentials.
2. Identify if the system enforces **account lockout** or delays further attempts.
3. Modify the **`X-Forwarded-For`** header to check if changing the IP address **resets the limit**.

📌 **Observation:** If changing the IP allows additional login attempts, the system **tracks rate limits based on IP rather than session or user ID**.

***

#### **Step 3: Configure Intruder for Username Enumeration**

1. Send the modified login request to **Burp Suite Intruder**.
2. Set **payload markers** on:
   * The **username** field.
   * The last digit of the **`X-Forwarded-For`** header.
3. Select **Pitchfork Attack Type** to test both variables simultaneously.

***

#### **Step 4: Measure Response Times for Enumeration**

1. Load a **username wordlist** (e.g., `/usr/share/seclists/Usernames/`).
2. In **Intruder > Options**, enable:
   * **Response Received**
   * **Response Completed**
3. Start the attack and **sort responses by timing**.

✅ **Valid usernames** → Noticeably **longer response times** (due to password hashing).\
❌ **Invalid usernames** → Immediate rejection (shorter response times).

***

#### **Step 5: Use a Long Password String for Confirmation**

1.  In the login request, replace the password with a **long string**, such as:

    ```plaintext
    testtesttesttesttesttesttesttesttesttesttesttesttesttesttesttesttest...
    ```
2. **Reasoning:**
   * If the **username is invalid**, the request is rejected instantly.
   * If the **username is valid**, the system hashes the password, increasing response time.

📌 **Observation:** A longer delay confirms **the username exists in the system**.

***

#### **Step 6: Password Enumeration**

1. Send the validated request to **Burp Suite Intruder**.
2. Configure **payload markers** for:
   * The **password** field.
   * The **`X-Forwarded-For`** IP (to bypass rate limits).
3. Load a **password list** and run the attack.
4. Look for **response anomalies**:
   * **302 Redirect or Success Status** → Correct password found.
   * **Different response time** → Partial password validation detected.

***

#### **Step 7: Log in with Discovered Credentials**

1. Attempt login using the **identified username and password**.
2. If **account lockout** is still active, **modify the `X-Forwarded-For` header** again.
3. **Intercept the login request** in Burp Suite, edit the necessary parameters, and **forward the request**.

✅ **Success Condition:** The system grants access.

***

### **Key Observations**

📌 **Timing Attacks Work Best When Password Hashing is Slow**

* Algorithms like **bcrypt and PBKDF2** amplify response time differences.
* Faster hash functions like **MD5 or SHA-1** reduce effectiveness.

📌 **Rate-Limiting Bypasses Rely on Header Manipulation**

* If **changing `X-Forwarded-For` resets the limit**, the system **relies on client-provided headers** rather than tracking unique sessions.

📌 **Intruder Sorting Helps Identify Timing Discrepancies**

* Sorting results by **response time** allows quick **username discovery**.

***

### **Next Steps**

🔹 I**mmediately check for:**

* **Rate-limiting mechanisms** (IP-based, session-based, or user-based).
* **Timing variations** to confirm **valid usernames**.
* **IP rotation or session resets** to bypass restrictions.

🚀 **Next Steps:**

* Experiment with **ffuf for faster enumeration**.
* Test similar attacks on **different authentication endpoints (e.g., API logins, admin panels)**.

***

This **lab subpage** provides a **step-by-step hands-on approach** to **response timing enumeration** and **rate-limit bypassing**, ensuring a **practical understanding** of the topic.

Would you like any refinements before finalizing? 🚀
