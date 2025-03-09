# 2FA Bypass (PortSwigger Lab)

#### **2FA Bypass via URL Manipulation – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **logic flaw** in a two-factor authentication (2FA) implementation, allowing an attacker to bypass the second authentication step and access a victim’s account. The attacker has already obtained valid login credentials but does not have the 2FA verification code. The goal is to exploit improper session management and authentication enforcement to gain access to **Carlos’s account** without completing the 2FA process.

* **Lab URL**: [PortSwigger 2FA Bypass Lab](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)
* **Credentials**:
  * Attacker: `wiener:peter`
  * Victim: `carlos:montoya`

***

### **Analysis and Exploitation**

#### **1. Understanding the 2FA Flow**

Before attempting a bypass, it was necessary to analyze how the application processes authentication and session management. By logging in with a valid account (`wiener:peter`), the following observations were made:

* **Initial login request (`/login`)**:
  * The request sends the **username and password**.
  * A **session cookie (`CookieA`)** is issued.
* **2FA request (`/login2`)**:
  * The request sends the **2FA code** received via email.
  * A new **session cookie (`CookieB`)** is issued after successful 2FA verification.

#### **2. Identifying the Vulnerability**

Using **Burp Suite**, session cookies were analyzed to determine whether the application enforced 2FA at the session level. The key finding was that **CookieA remained valid** after the initial login, even before completing the 2FA step. This suggested that session management did not fully enforce 2FA before allowing access to authenticated pages.

#### **3. Exploiting the Flaw**

1. **Log in with the victim’s credentials (`carlos:montoya`)** until reaching the 2FA verification step.
2. **Modify the URL manually** to access a protected page (`/my-account`) without providing a 2FA code.
3. If successful, the page loads, confirming that **the backend does not enforce 2FA verification before granting access to sensitive user data**.

#### **4. Why This Works**

The vulnerability exists because the server:

* Fails to enforce 2FA validation before granting access to protected pages.
* Does not invalidate the session established after the first login step (`CookieA`).
* Assumes users will always complete the 2FA step before proceeding to other areas of the application.

#### **5. Key Takeaways**

* **Session cookies should be invalidated until 2FA is fully completed.**
* **Multi-step authentication must be enforced on all protected routes.**
* **Manual URL navigation should not allow users to bypass authentication steps.**

***

### **Security Best Practices to Prevent This Flaw**

#### **1. Implement Strong Session Management**

* **Invalidate initial session tokens** after a successful login but before 2FA verification.
* **Issue a new session token only after completing the full authentication process.**

#### **2. Restrict Direct URL Access**

* Ensure that **all sensitive pages require 2FA validation** before granting access.
* Apply **server-side access controls** to prevent unauthorized navigation to restricted areas.

#### **3. Monitor and Log Authentication Attempts**

* Log **failed 2FA attempts** and alert administrators about unusual login behaviors.
* Implement **rate-limiting** to prevent brute-force attempts on authentication mechanisms.

***

### **Final Thoughts**

This lab highlights how **poorly enforced session management** can undermine the security benefits of multi-factor authentication. Logical vulnerabilities, such as **bypassing authentication steps via direct URL navigation**, can allow attackers to bypass intended security controls. Identifying and addressing these issues is critical for ensuring that MFA implementations provide **effective security** rather than a **false sense of protection**.
