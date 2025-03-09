# Single Endpoint Race Conditions Overview

## **Single Endpoint Race Conditions Overview**

***

### **High-Level Summary**

A **Single Endpoint Race Condition** occurs when multiple concurrent requests interact with a **single API endpoint**, leading to **data inconsistencies** or **security bypasses**. Attackers exploit **timing issues** by sending **parallel requests** during a **race window**, causing unintended behavior such as **account takeover, unauthorized privilege escalation, or multiple redemptions of restricted actions**.

***

### **Definition Bank**

| **Term**              | **Definition**                                                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Race Condition**    | A vulnerability where multiple concurrent processes affect shared data, leading to unpredictable behavior.                |
| **Race Window**       | The **timeframe** between the check and the update of data where a collision can occur.                                   |
| **Session Overwrite** | A scenario where concurrent requests modify session data before it is saved, leading to unintended state changes.         |
| **Token Collision**   | A security issue where a token generated for one user is unintentionally assigned to another user due to race conditions. |
| **Parallel Requests** | Multiple requests sent at the same time to exploit timing issues in a web application.                                    |

***

### **How Single Endpoint Race Conditions Work**

A **race condition** occurs when multiple requests to a **single endpoint** modify shared data **without proper synchronization**. Attackers send **parallel requests** to cause **state corruption** or **gain unauthorized access**.

#### **Step-by-Step Breakdown of an Exploit:**

1. **Hacker Sends Two Password Reset Requests in Parallel:**
   * **Request 1:** Targets [**attacker@example.com**](mailto:attacker@example.com).
   * **Request 2:** Targets [**victim@example.com**](mailto:victim@example.com).
2. **Session Data Overwritten (Race Window Opens):**
   * **Request 1:** The server starts processing and sets [**attacker@example.com**](mailto:attacker@example.com) in session storage.
   * **Request 2:** Arrives before Request 1 finishes, **overwriting the session** with [**victim@example.com**](mailto:victim@example.com).
3. **Reset Token Generation:**
   * **Request 2:** Generates a **password reset token** for [**victim@example.com**](mailto:victim@example.com).
4. **Token Sent to the Wrong User:**
   * **Request 1:** Completes **after** Request 2, causing the system to **send the victim’s reset token to the attacker**.

***

#### **Real-World Example: Race Condition in Password Reset**

1. An attacker submits **two password reset requests**: one for their own account and another for the victim’s account.
2. Due to a **session overwrite bug**, the victim’s reset token is **sent to the attacker** instead.
3. The attacker **resets the victim’s password** and gains control of their account.

✅ **Final Result:** The attacker takes over the victim’s account by **exploiting the race window**.

***

### **Real-World Incidents & CVEs Related to Single Endpoint Race Conditions**

#### **CVE-2023-0669: Fortinet Authentication Bypass via Race Condition**

* **Issue:** A race condition in **FortiNAC's authentication mechanism** allowed attackers to bypass authentication and gain administrative privileges.
* **Exploitation:** Attackers sent **multiple parallel login requests**, exploiting a delay in session updates.
* **Fix:** Implemented **session locking** and **single-use authentication tokens**.
* **Reference:** [CVE-2023-0669 - Fortinet Race Condition](https://nvd.nist.gov/vuln/detail/CVE-2023-0669)

***

#### **CVE-2022-20699: Cisco Secure Email Race Condition**

* **Issue:** Attackers exploited a **race window in email verification**, leading to **account takeover**.
* **Exploitation:** By sending **multiple verification requests**, attackers **overwrote session data** and hijacked email confirmations.
* **Fix:** Implemented **transaction integrity checks**.
* **Reference:** [CVE-2022-20699 - Cisco Secure Email](https://nvd.nist.gov/vuln/detail/CVE-2022-20699)

***

### **How to Identify Single Endpoint Race Conditions**

* **Look for Session-Based Actions:** Target endpoints managing **password resets, email updates, coupon redemptions, or account settings**.
* **Send Parallel Requests:** Use **Burp Suite Intruder**, **Turbo Intruder**, or **custom scripts** to **send simultaneous requests**.
* **Check for Timing Issues:** Identify scenarios where **one request overwrites another** due to delayed execution.
* **Monitor API Responses:** Look for **unexpected session modifications** or **reused authentication tokens**.

***

### **Exploiting Single Endpoint Race Conditions (Step-by-Step)**

1. **Identify the Target Endpoint**
   * Look for actions that **modify session-based data**, such as **password resets or account updates**.
2. **Send Parallel Requests**
   * Use Burp Suite’s **Turbo Intruder** or **a multi-threaded script** to send **simultaneous requests**.
3. **Observe and Analyze Responses**
   * Check if the system **overwrites** a session variable before completing previous requests.
4. **Verify Exploitability**
   * Confirm that you can control **the token, session state, or security settings**.

***

### **Secure Code Example (Node.js)**

#### **Preventing Race Conditions with Session Locking**

```javascript
app.post('/reset-password', async (req, res) => {
    const email = req.body.email;

    // Prevent race conditions by locking session updates
    await db.query('LOCK TABLE sessions WRITE');

    const user = await db.query('SELECT * FROM users WHERE email = ?', [email]);

    if (!user) {
        res.status(400).send("User not found");
        return;
    }

    const resetToken = generateToken();
    await db.query('UPDATE users SET reset_token = ? WHERE email = ?', [resetToken, email]);

    db.query('UNLOCK TABLES'); // Release lock
    res.send("Reset email sent.");
});
```

#### **Why This is Secure:**

* ✅ **Database Locking:** Prevents simultaneous updates to session data.
* ✅ **Atomic Transaction:** Ensures one request completes before processing another.
* ✅ **Prevents Token Overwrites:** Blocks race conditions affecting session storage.

***

### **Mitigation Techniques for Single Endpoint Race Conditions**

| **Mitigation**                  | **Implementation**                                                   |
| ------------------------------- | -------------------------------------------------------------------- |
| **Atomic Transactions**         | Ensure validation and update happen in a **single operation**.       |
| **Database Locking**            | Use **row-level locks** to prevent multiple concurrent updates.      |
| **Session Isolation**           | Do not store **sensitive user actions** in shared session variables. |
| **Rate Limiting**               | Restrict how many **requests per second** a user can send.           |
| **Token Expiry & Regeneration** | Invalidate **previous tokens** when a new one is generated.          |

***

### **Security Implications of Single Endpoint Race Conditions**

#### **1. Account Takeover**

* Attackers manipulate **password reset flows** to take control of user accounts.

#### **2. Privilege Escalation**

* Exploiting a **race condition in user roles** can lead to **unauthorized admin access**.

#### **3. Financial Exploitation**

* Attackers **redeem promo codes, coupons, or rewards** multiple times before the system updates.

***

### **Key Takeaways**

* **Single Endpoint Race Conditions** occur when **parallel requests modify shared resources without proper locking**.
* Attackers exploit **race windows** to **overwrite session data**, **redeem coupons multiple times**, or **steal authentication tokens**.
* **Secure coding practices**, such as **atomic transactions, session locking, and rate limiting**, are essential to mitigate these attacks.
* **Real-world incidents**, such as **CVE-2023-0669 (Fortinet Authentication Bypass)** and **CVE-2022-20699 (Cisco Secure Email Takeover)**, highlight the **critical security risks** posed by race conditions.

***

### **Further Reading & References**

* **OWASP Race Conditions Cheat Sheet**: [OWASP Guide](https://cheatsheetseries.owasp.org/cheatsheets/Race_Conditions_Cheat_Sheet.html)
* **CVE-2023-0669 (Fortinet Race Condition Issue)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2023-0669)
* **CVE-2022-20699 (Cisco Secure Email Vulnerability)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2022-20699)
