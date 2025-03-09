# Multi-step process with no access control on one step (PortSwigger Lab)

## **Exploiting Multi-Step Process with Missing Access Controls – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **multi-step access control vulnerability** where a critical action—**promoting a user to an administrator**—is not properly validated at every step. Instead of verifying **both user identity and role**, the application allows an attacker to replay an **administrative request** while substituting a **non-admin user’s session cookie**, effectively granting administrative privileges without proper authorization.

* **Lab URL**: [PortSwigger Multi-Step Process Lab](https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step)
* **Target:** Use an **access control flaw** to **promote Wiener to an admin** by modifying a captured admin request.

***

### **Analysis and Exploitation**

#### **1. Understanding the Vulnerability**

* **Why is this an access control issue?**
  * The application **fails to enforce role-based access control (RBAC)** at every step of the admin promotion process.
  * The **promotion request** does not require a validated **admin session**, meaning an attacker can replay a valid request and substitute their own session.
  * **Session cookie manipulation** allows a low-privileged user to escalate their privileges.
* **Why is this critical?**
  * **Administrative actions should always validate user identity and role** before being processed.
  * Attackers can **exploit weak access controls in multi-step processes** to escalate privileges or perform unauthorized actions.

***

#### **2. Exploiting the Vulnerability**

**Step 1: Capture the Admin Promotion Request**

1. Log in using the **administrator credentials** (`administrator:admin`).
2. Navigate to the **Admin Panel** and promote Carlos to an administrator.
3.  Capture the **POST request to /admin-roles** in Burp Suite:

    ```plaintext
    POST /admin-roles HTTP/2
    Host: <lab-url>
    Cookie: session=admin_session_cookie
    Content-Type: application/x-www-form-urlencoded

    action=upgrade&confirmed=true&username=carlos
    ```
4. **Downgrade Carlos back to a normal user** to ensure the request remains reusable.
5. Send the captured request to **Burp Suite Repeater**.

***

**Step 2: Capture Wiener’s Session Cookie**

1. Open a **private/incognito browser window** and log in as Wiener (`wiener:peter`).
2. Use **Burp Suite Proxy** to capture Wiener’s **session cookie** from a request.
3.  Extract Wiener’s session cookie from a request such as:

    ```plaintext
    GET /my-account?id=wiener HTTP/2
    Cookie: session=wiener_session_cookie
    ```
4. Copy Wiener’s session cookie for use in the modified admin request.

***

**Step 3: Modify and Replay the Promotion Request**

1.  In **Burp Suite Repeater**, replace the **admin session cookie** with **Wiener’s session cookie**:

    ```plaintext
    Cookie: session=wiener_session_cookie
    ```
2.  Modify the **username parameter** to promote Wiener instead of Carlos:

    ```plaintext
    action=upgrade&confirmed=true&username=wiener
    ```
3. Send the modified request.

***

**Step 4: Confirm the Exploit Worked**

1. Observe the **HTTP/2 302 Found** response, indicating that the promotion request was successfully processed.
2. **If redirected to a 401 Unauthorized page, ignore it**—the promotion has already occurred.
3. Log out and **log back in as Wiener** (`wiener:peter`).
4. Check Wiener’s role in the lab interface—Wiener is now an **administrator**, completing the lab.

***

### **Why This Works (Security Breakdown)**

* The **server does not validate** whether the session initiating the request belongs to an administrator.
* **Session cookies are misused**—the system allows any valid session to send an admin request.
* **Role-based access control (RBAC) is missing**, allowing unauthorized privilege escalation.

***

### **Key Takeaways**

#### **1. Multi-Step Processes Require Consistent Access Control**

* **Every step** of a sensitive workflow should verify user **identity and role** before processing.
* Authorization should be checked **before executing administrative actions**, not just during initial authentication.

#### **2. Session Cookies Must Be Properly Scoped**

* A session cookie should not grant admin privileges simply by replaying an admin request.
* **Each request should require explicit privilege validation.**

#### **3. Mitigating Multi-Step Access Control Vulnerabilities**

**Implement Server-Side Role Verification**

* Ensure that **all administrative actions validate the requesting user’s role**.
* Enforce **strict privilege validation at each step** of a multi-step process.

**Tie Actions to the User Who Initiates Them**

* Instead of allowing any session to send an admin request, enforce **session-bound user verification**.

**Prevent Cookie Reuse for Privilege Escalation**

* Implement **short-lived session tokens** for critical actions.
* Log and **monitor unusual session activity**, such as **privilege escalations from non-admin accounts**.

***

### **Final Thoughts**

This lab highlights **how missing access control enforcement in multi-step processes can lead to privilege escalation**. Ensuring **consistent authorization checks across all workflow steps** is critical for securing administrative actions.

By implementing **server-side role validation, session scoping, and privilege enforcement**, organizations can prevent **unauthorized privilege escalation attacks** and **eliminate weak access control vulnerabilities**.
