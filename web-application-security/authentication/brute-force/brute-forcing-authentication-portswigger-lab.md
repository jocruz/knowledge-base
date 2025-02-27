# Brute Forcing Authentication (PortSwigger Lab)

***

## **Lab - Brute Forcing Authentication**

### **High-Level Summary**

This lab explores brute force techniques to identify valid username and password combinations. By leveraging tools like **Burp Suite** and **ffuf**, we analyze authentication mechanisms, detect enumeration vulnerabilities, and perform password attacks. Additionally, we discuss **remediation strategies** to prevent such attacks in real-world applications.

***

### **Tools Used**

* **Burp Suite** – Intercepting requests, analyzing responses, and automating brute-force attacks.
* **ffuf** – Fast fuzzing and brute-force automation for authentication testing.

***

### **Objective**

Use brute force techniques to identify valid username and password combinations through common attack methods.

[**Guided Lab: Brute Force Attacks**](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

***

### **Lab Steps**

#### 1. **Initial Login Attempt**

* Navigate to **My Account** and attempt a login using invalid credentials.
* Observe the **error message** returned (e.g., _"Invalid Username"_).
* Note: The **username and password fields** are both **8 characters long** (potential length restriction).

***

#### 2. **Analyze the POST Request**

* Intercept the login request using **Burp Suite**.
*   Locate the `POST /login` request and examine the request body:

    ```plaintext
    username=test&password=test
    ```
* Identify any **distinctive responses** that could help enumerate valid usernames.

***

#### 3. **Username Enumeration with Burp Intruder**

* Set **Burp Intruder** to target the `username` field.
* Load a **username wordlist** (e.g., `/usr/share/seclists/Usernames/`).
* Start the attack and analyze responses:
  * **Success Indicator**: Look for **response content length differences** (e.g., `3250` vs. `187`).
  * **Common Response**: `"Invalid username"` → Username does not exist.
  * **Different Response**: `"Incorrect password"` → Username is valid.
* Use the **Render tab** in Burp Suite to confirm responses.

***

#### 4. **Password Brute Force**

* After identifying a valid username, repeat the process for the `password` field.
* Look for a **significantly different response length** when a valid password is attempted.
* Example expected behavior:
  * Invalid credentials → **Content-Length: 187**
  * Valid credentials → **Content-Length: 3250**

***

#### 5. **Automated Brute Force with ffuf**

*   Save the `POST` request into a file:

    ```plaintext
    username=FUZZUSER&password=FUZZPASS
    ```
*   Execute `ffuf` in **clusterbomb mode** for username and password guessing:

    ```bash
    ffuf -request req.txt -request-proto https -mode clusterbomb \
    -w /home/kali/usernames.txt:FUZZUSER \
    -w /home/kali/passwords.txt:FUZZPASS -mc 302
    ```
* Example successful credentials discovered:
  * **Username:** `adkit`
  * **Password:** `michelle`

***

### **Observations**

* **Content-Length variations** are the primary indicator of authentication status (`3250` vs. `187`).
* Use **negative search filters** in Burp Suite to **exclude known invalid responses** for efficiency.
* Brute force may be limited by **rate-limiting or account lockouts**—testing for these is essential.

***

### **Remediation Strategies**

#### **1. Implement Account Lockout & Delays**

* **Enforce lockouts** after multiple failed login attempts.
* Implement **progressive delays** (e.g., increasing wait times per failed attempt).

#### **2. Require Strong Authentication Methods**

* **Enforce MFA (Multi-Factor Authentication)** to prevent single-factor brute force attacks.
* Require **password complexity rules** (longer, unique passwords).

#### **3. Prevent Username Enumeration**

* **Generic error messages** (e.g., _"Invalid credentials"_) instead of revealing **"Invalid username" vs. "Incorrect password."**
* Normalize response times for **valid and invalid usernames** to prevent response time-based enumeration.

#### **4. Implement Rate-Limiting & Logging**

* **Restrict login attempts** per IP per minute.
* **Monitor login logs** for excessive failed attempts and trigger alerts.

***

### **Real-World Application**

* **Bug Bounties**: Username enumeration remains a common vulnerability in web applications, often reported in security programs.
* **Penetration Testing**: Brute force attacks are an essential component of **web application security assessments**.
* **Blue Team / SOC Monitoring**: Security teams detect and mitigate brute force attacks by **monitoring logs and implementing defenses**.

***
