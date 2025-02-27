---
description: >-
  This section provides a deeper look into response time enumeration techniques,
  explaining how attackers use authentication delays to extract information
  about valid usernames and bypass security measu
---

# Response Time Enumeration (Concepts & Techniques)

## **Concept Overview**

Response time enumeration exploits **inconsistencies in server response times** to infer authentication details. This method is useful for:

* **Username enumeration** – Detecting valid usernames based on login processing delays.
* **Rate limit analysis** – Identifying misconfigured or bypassable request throttling.

Attackers analyze **response latency** to determine whether a request is processed differently for valid vs. invalid credentials.

***

### **Key Concepts**

#### **1. Authentication Timing Analysis**

**How Response Timing Works**

When credentials are submitted, the server follows this process:

| **Scenario**                        | **System Behavior**                                  | **Expected Response Time** |
| ----------------------------------- | ---------------------------------------------------- | -------------------------- |
| **Invalid username**                | Immediate rejection (password check skipped).        | **Faster response**        |
| **Valid username + incorrect pass** | Password hashing occurs before rejection.            | **Slight delay**           |
| **Valid username + long password**  | Hashing takes longer, causing an even greater delay. | **Longest response time**  |

**Key Observations**

* **Slower hashing algorithms** (e.g., **bcrypt, PBKDF2, Argon2**) increase timing discrepancies.
* **Longer passwords = longer response times** → Attackers use this to **confirm valid usernames**.
* Timing differences **reveal authentication workflows**, exposing security weaknesses.

**Practical Use Case**

Attackers submit:

* **Invalid usernames** → Expect **instant response**.
* **Valid usernames with long passwords** → Expect **delayed response**.
* **Analysis of timing differences** allows attackers to **map valid usernames**.

***

### **Example Scenario**

**1. Submit login requests:**

* **Username:** `johndoe`
* **Password:** `password123`
* **Observation:** Response time is **fast** → Username may be invalid.

**2. Submit with a long password:**

* **Username:** `johndoe`
* **Password:** `superlongpasswordthatisverysecure123456789`
* **Observation:** Response time is **slower** → Username likely exists.

***

### **Tools & Techniques**

#### **Burp Suite for Response Time Analysis**

**1. Using Repeater**

* Monitor response times in the **bottom-right corner** of Burp Suite.
* Compare response times across **valid and invalid usernames**.

**2. Using Intruder**

* Set payload markers for **username fuzzing**.
* Add columns for:
  * **Response Received**
  * **Response Completed**
* Identify **slower responses** that indicate valid usernames.

**3. Using ffuf (Optional for Automation)**

*   Automate testing with `ffuf` to measure response time variations:

    ```bash
    ffuf -u https://target.com/login -X POST -d "username=FUZZ&password=test" \
    -w /home/kali/usernames.txt -mc 200 -ac -t 50
    ```

    * **-mc 200** → Match HTTP 200 responses.
    * **-ac** → Automatically calibrate responses.
    * **-t 50** → Use 50 concurrent threads.

***

### **Bypassing Rate Limits Using Response Timing**

#### **1. Rate Limiting Basics**

Rate limiting restricts repeated login attempts by tracking:

* **IP addresses**
* **Session tokens**
* **User agents**

#### **2. Common Bypass Techniques**

| **Bypass Method**              | **How It Works**                                       |
| ------------------------------ | ------------------------------------------------------ |
| **IP Address Rotation**        | Modify `X-Forwarded-For`, `X-Real-IP`, or `Client-IP`. |
| **Session Token Manipulation** | Reset cookies/session tokens after failed attempts.    |
| **User-Agent Rotation**        | Change browser/device identifiers.                     |
| **Slow Brute Force**           | Reduce request rate to avoid detection.                |

***

### **Practical Steps for Enumeration**

#### **Step 1: Identify the Login Endpoint**

* Locate the `POST /login` request.
*   Inspect the request body:

    ```plaintext
    username=test&password=test
    ```

#### **Step 2: Measure Response Times**

* Use Burp Suite **Repeater** to send multiple login requests.
* Observe response time differences **for valid vs. invalid usernames**.

#### **Step 3: Configure Intruder for Username Fuzzing**

1. Send the login request to **Intruder**.
2. Add a **payload marker** around the `username` field.
3. Use a **username wordlist** (e.g., `/usr/share/seclists/Usernames/`).
4. Enable **Response Timing Analysis** in the **Results tab**.
5. Identify **longer response times** → These indicate valid usernames.

***

### **Next Steps**

#### **Checklist**

✅ **Monitor Response Timing Differences** → Validate findings with repeated tests.\
✅ **Rate Limiting Testing** → Identify **tracking mechanisms** (IP, cookies, headers).\
✅ **Troubleshooting** → Consider network latency & adjust testing methodology.

#### **Next Steps**

* **Once valid usernames are identified**:
  * Proceed with **password brute forcing**.
  * Explore **credential stuffing attacks**.
  * Investigate additional enumeration techniques.
