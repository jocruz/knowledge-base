# Weak Access Control

#### **Weak Access Controls – Overview**

**What Are Weak Access Controls?**

Weak access controls occur when an application fails to **consistently enforce authorization restrictions**, often allowing users to access or manipulate resources beyond their intended permissions. These vulnerabilities typically arise due to **inconsistent enforcement of access policies**, reliance on **user-controlled inputs** (such as HTTP methods or headers), or improperly implemented authentication checks.

Weak access controls can lead to **unauthorized access to sensitive data**, **privilege escalation**, and **manipulation of critical application functions**.

***

### **Key Takeaways**

#### **1. Common Causes of Weak Access Controls**

* **User-Supplied Inputs**
  * Applications may trust user-controllable inputs for authorization checks, such as:
    * **HTTP Methods** (e.g., switching from `GET` to `POST`).
    * **Headers** (e.g., `Referer`, `X-Origin-URL`).
* **Missing or Inconsistent Access Controls**
  * Some application workflows enforce access controls at the **initial login** but fail to enforce them at **critical actions** such as modifying user data or deleting accounts.

#### **2. Testing for Weak Access Controls**

1. **Compare Requests Across User Roles**
   * Log in as an **admin** and perform various actions.
   * Replay the same requests with a **low-privileged user’s session**.
   * Analyze whether the low-privileged user can perform **restricted actions**.
2. **Test for Header Manipulation**
   * Modify or remove **security-related headers** (e.g., `Referer`, `X-Origin-URL`).
   * Observe whether the application still restricts access correctly.
3. **Analyze HTTP Methods**
   * Test if changing the HTTP method (`GET`, `POST`, `PUT`, `DELETE`) bypasses access restrictions.

***

#### **Example: Weak Access Control Flow**

1. **Login as Admin**
   * Access controls **enforced**.
   * Security headers are present.
2. **Access Admin Panel**
   * Access controls **enforced**.
   * Security headers are present.
3. **Perform Admin Action (Delete User)**
   * **Access controls missing**.
   * The request can be **replayed by a low-privileged user**.

***

### **Common Weaknesses in Access Controls**

#### **1. Lack of Role-Based Authorization Checks**

* Applications only check **if a user is authenticated**, but fail to verify if they have **proper privileges**.
* Example: Any logged-in user can access `/admin/delete-user?id=123`.

#### **2. Header-Based Authentication Bypass**

* The application relies on `Referer` or `X-Origin-URL` headers for access control.
* Attackers can **modify or remove headers** to bypass restrictions.

#### **3. HTTP Method Manipulation**

* The application restricts certain actions based on **request type**, rather than verifying user roles.
* Example: The frontend only provides a "Delete" button for admins, but an attacker can manually send a `DELETE` request from a non-admin account.

***

### **Best Practices for Preventing Weak Access Controls**

#### **1. Enforce Access Controls at the API and Backend Level**

* Do not rely solely on **client-side checks or UI restrictions**.
* Validate **both authentication and user roles** before processing sensitive requests.

#### **2. Implement Role-Based Access Control (RBAC)**

* Assign **granular permissions** to user roles (e.g., Admin, Moderator, User).
* Enforce **server-side role validation** for each request.

#### **3. Avoid Using Headers for Authentication**

* Do not trust `Referer`, `X-Forwarded-For`, or similar headers for access control.
* Implement **token-based authentication mechanisms** instead.

#### **4. Validate HTTP Methods Strictly**

* Restrict actions based on **user roles, not just request methods**.
* Ensure that **`DELETE` and `PUT` requests require proper authentication**.

#### **5. Audit and Monitor Access Logs**

* Detect **unusual access attempts** such as:
  * **Multiple requests from a low-privileged user to admin endpoints**.
  * **Requests missing standard security headers**.

***

### **Testing Weak Access Controls**

#### **1. Compare Requests Across User Roles**

* Capture admin requests using **Burp Suite**.
* Replay the same requests using a **low-privileged session**.
* Observe whether access is incorrectly granted.

#### **2. Modify HTTP Headers**

* Remove or modify `Referer`, `X-Origin-URL`, or `X-Forwarded-For`.
* Observe if access control can be bypassed.

#### **3. Test HTTP Method Bypass**

* Change `GET` to `POST` or `PUT` and resend the request.
* Check if the application allows unauthorized modifications.

***

### **Final Thoughts**

Weak access controls are a **major security risk** that can lead to **privilege escalation, data breaches, and unauthorized system modifications**. Ensuring **consistent access control enforcement across all application workflows** is critical for securing sensitive operations.

By implementing **server-side authorization, role-based access controls, and strict validation of request methods and headers**, organizations can mitigate **common access control vulnerabilities** and prevent unauthorized access to sensitive resources.
