# Referer-based access control (PortSwigger Lab)

## **Exploiting Referer-Based Access Control – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **weak access control mechanism** that relies on the **Referer header** to restrict access to administrative functionality. Because the Referer header is **client-controlled**, an attacker can **bypass access restrictions** by modifying the request and replacing the session cookie.

* **Lab URL**: [PortSwigger Referer-Based Access Control Lab](https://portswigger.net/web-security/access-control/lab-referer-based-access-control)
* **Target:** Exploit the Referer-based access control mechanism to **promote a non-admin user (Wiener) to an administrator** by replaying a captured admin request with a different session.

***

### **Analysis and Exploitation**

#### **1. Understanding the Vulnerability**

* **Why is this an access control issue?**
  * The application **relies on the Referer header** to determine whether a user has permission to access administrative functionality.
  * Attackers can **omit or manipulate the header** to bypass restrictions.
  * **Session cookies are not properly validated**, allowing an attacker to escalate privileges by replaying an admin request.
* **Why is this critical?**
  * Relying on **client-controlled headers** for security is **inherently flawed**, as they can be modified by an attacker.
  * **Server-side access control** should be implemented to properly enforce authorization for sensitive actions.

***

#### **2. Exploiting the Vulnerability**

**Step 1: Capture the Admin Promotion Request**

1. Log in using the **administrator credentials** (`administrator:admin`).
2. Navigate to the **Admin Panel** and promote Carlos to an administrator.
3.  Capture the **GET request to /admin-roles** in Burp Suite:

    ```plaintext
    GET /admin-roles?username=carlos&action=upgrade HTTP/2
    Host: <lab-url>
    Referer: https://<lab-url>/admin
    Cookie: session=admin_session_cookie
    ```
4. **Downgrade Carlos back to a normal user** to ensure the request remains reusable.
5. Send the captured **promotion request** to **Burp Suite Repeater**.

***

**Step 2: Remove or Modify the Referer Header**

1.  In Burp Suite Repeater, **delete the Referer header** or modify it to a non-admin page:

    ```plaintext
    Referer: https://<lab-url>/home
    ```
2. Send the modified request.
3. If the response is **HTTP 401 Unauthorized**, proceed to the next step.

***

**Step 3: Capture Wiener’s Session Cookie**

1. Open a **private/incognito browser window** and log in as Wiener (`wiener:peter`).
2.  Use **Burp Suite Proxy** to capture Wiener’s **session cookie** from a request:

    ```plaintext
    GET /my-account?id=wiener HTTP/2
    Cookie: session=wiener_session_cookie
    ```
3. Copy Wiener’s session cookie for use in the modified admin request.

***

**Step 4: Modify and Replay the Promotion Request**

1.  In **Burp Suite Repeater**, replace the **admin session cookie** with **Wiener’s session cookie**:

    ```plaintext
    Cookie: session=wiener_session_cookie
    ```
2.  Modify the **username parameter** to promote Wiener instead of Carlos:

    ```plaintext
    GET /admin-roles?username=wiener&action=upgrade HTTP/2
    ```
3. Send the modified request.
4. Observe the **HTTP 302 Found** response, indicating that the promotion request was processed.
5. If redirected to **HTTP 401 Unauthorized**, ignore it—the promotion has already occurred.

***

**Step 5: Verify Success**

1. Log out and **log back in as Wiener** (`wiener:peter`).
2. Check Wiener’s role in the lab interface—Wiener is now an **administrator**, completing the lab.

***

### **Why This Works (Security Breakdown)**

* The **Referer header is client-controlled** and can be **manipulated or omitted**.
* The application **does not enforce server-side validation** of user roles for administrative actions.
* **Session cookies are not properly scoped**, allowing attackers to replay admin requests.

***

### **Key Takeaways**

#### **1. The Referer Header is Not a Security Control**

* The **Referer header can be modified** or removed entirely, making it an unreliable security mechanism.
* **Server-side validation** should be enforced to determine if the user has the correct permissions.

#### **2. Access Control Should Be Enforced on the Server**

* Each **administrative action** should verify **both the user’s identity and role** before being executed.
* Role-based access control (RBAC) should be implemented to prevent unauthorized privilege escalation.

#### **3. Mitigating Referer-Based Access Control Vulnerabilities**

* **Enforce strict role validation on the server** for all administrative actions.
* **Do not rely on client-side headers** to determine authorization.
* **Log and monitor administrative actions** to detect suspicious behavior.

***

### **Using Firefox Multi-Account Containers for Testing**

#### **What are Multi-Account Containers?**

* **Firefox Multi-Account Containers** allow testers to maintain **separate browser sessions** in different tabs.
* Each container acts as an **isolated browser instance**, preventing cookie sharing between tabs.

#### **How to Use Them for Access Control Testing**

1. Install the **Firefox Multi-Account Containers** extension.
2. Open different tabs and assign them to separate containers (e.g., one for admin, one for user).
3. Log into the web application with different user roles in each container.
4. Test **role escalation vulnerabilities** by replaying captured requests with different session cookies.

***

### **Weak Access Controls in the Lab**

#### **Access Control Mechanism**

* The application relies on the **Referer header** to restrict access to administrative functionality.

#### **Why This is Weak**

* The **Referer header is user-controlled** and can be removed or manipulated.
* The server **does not validate the user’s role**, allowing unauthorized role escalation.

#### **Mitigation Strategies**

* Implement **server-side role verification** for all administrative actions.
* Do not rely on **client-provided headers** for authorization.

***

### **Final Thoughts**

This lab highlights a **critical access control vulnerability** caused by relying on **client-controlled headers** instead of proper **server-side role validation**. Attackers can easily bypass **Referer-based access control** to **escalate privileges** and **gain unauthorized administrative access**.

By enforcing **strict role-based access control**, **validating session tokens properly**, and **avoiding client-side headers for security decisions**, applications can prevent **unauthorized access and privilege escalation attacks**.
