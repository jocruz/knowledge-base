# Insecure direct object references (PortSwigger Lab)

## **Exploiting IDOR in Live Chat Transcripts – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability in a **live chat transcript feature**. The vulnerability allows unauthorized users to retrieve another user's **sensitive information (password)** by **enumerating transcript file names**. By exploiting **sequential file references**, an attacker can obtain **Carlos’s password** and gain unauthorized access to his account.

* **Lab URL**: [PortSwigger IDOR Lab - Live Chat Transcripts](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references)
* **Target:** Extract Carlos’s password from a leaked chat transcript and log in to his account.

***

### **Analysis and Exploitation**

#### **1. Understanding the IDOR Vulnerability**

* **Why is this an IDOR?**
  * The application **fails to enforce proper access control** on chat transcript files.
  * Users can **enumerate transcript file names** (e.g., `2.txt → 1.txt → 0.txt`) to access **other users' transcripts**.
  * No authentication or session validation is required to retrieve another user's transcript.
* **Why is this critical?**
  * **Predictable object references** (like `2.txt`, `3.txt`) allow attackers to **enumerate and access unauthorized data**.
  * If **transcripts contain sensitive information** (like passwords), attackers can escalate the issue into **full account takeover**.

***

#### **2. Exploiting the Vulnerability**

**Step 1: Generate a Chat Transcript**

1. Navigate to the **live chat feature** and send a message:
   * **Example:** "Hello there."
2. After sending the message, click **View Transcript**.
3. Observe the transcript **file URL** (e.g., `/download-transcript/2.txt`).

**Step 2: Investigate Transcript Access**

1. Identify the **pattern in transcript file names** (e.g., they use **incrementing numbers**).
2. Send the **GET /download-transcript/2.txt** request to **Burp Repeater** for testing.

**Step 3: Enumerate Transcript Files**

1. Modify the file name in Burp Suite Repeater:
   * Change `2.txt` to `1.txt` and send the request.
2. If a **valid transcript** is found, review its contents for **sensitive information**.
3. Automate enumeration with **Burp Intruder**:
   * Send the request to **Intruder**.
   * Set a **positional marker** on the file number (`/download-transcript/{2}.txt`).
   * Use **Numbers payload type** with a range of **0-50** to cycle through file numbers.
4. Start the attack and analyze responses:
   * **`200 OK`** → Valid transcript found.
   * **`400 Bad Request`** → No transcript available.

***

**Step 4: Extract Sensitive Information**

1. Review the **200 OK responses** for **leaked credentials**.
2. Example of a leaked password in a chat transcript:
   * **"The password for Carlos is SuperSecure123."**

***

**Step 5: Log In with Stolen Credentials**

1. Navigate to the login page.
2. Enter **Carlos’s credentials**:
   * **Username:** `Carlos`.
   * **Password:** `SuperSecure123` (retrieved from the transcript).
3. Access Carlos’s account to complete the lab.

***

### **Why This Works (Security Breakdown)**

* **No access control validation** prevents users from retrieving another person’s transcript.
* The system uses **sequential identifiers** (e.g., `1.txt`, `2.txt`), making enumeration easy.
* **Sensitive data (passwords) is stored in an easily accessible format**, creating a severe security risk.

***

### **Key Takeaways**

#### **1. Why IDOR is Dangerous in Real Applications**

* **Exposing predictable object references** enables **mass enumeration attacks**.
* **Personal data, passwords, and authentication tokens** stored in exposed objects can lead to **account takeovers**.
* Attackers can **harvest sensitive data at scale** with automated tools.

#### **2. How to Prevent This Vulnerability**

**Implement Proper Access Controls**:

* Restrict access to **transcript files based on session authentication**.
* Require users to be logged in and **validate ownership** before serving transcripts.

**Use Secure, Non-Predictable Object References**:

* Use **randomized UUIDs** instead of sequential numbers for file access (e.g., `download-transcript/d8a9f6b2.txt`).

**Limit Sensitive Data Exposure**:

* Avoid storing **passwords, security questions, or personally identifiable information (PII)** in chat logs or transcripts.

**Monitor and Log Unusual Access Attempts**:

* Detect **multiple sequential requests** for transcript files (possible enumeration).

***

### **Final Thoughts**

This lab demonstrates **how predictable object references without access control can be exploited** to gain unauthorized access to **sensitive user data**. **IDOR remains one of the most common web security flaws**, often leading to severe data breaches.

By enforcing **server-side validation, implementing secure access controls, and using non-sequential identifiers**, applications can prevent **unauthorized access to user resources** and **protect sensitive information from exposure**.
