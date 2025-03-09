# User ID controlled by request parameter (PortSwigger Lab)

## **Exploiting ID-Based Access Control – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **Broken Function Level Authorization (BFLA)** vulnerability, where an attacker can manipulate the `id` parameter in the URL to access another user's sensitive information, specifically their **API key**. The goal is to bypass authorization mechanisms by modifying request parameters and retrieving **Carlos’s API key**.

* **Lab URL**: [PortSwigger User ID Controlled by Request Parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)
* **Credentials**:
  * Attacker: `wiener:peter`
  * Victim: `carlos`

***

### **Analysis and Exploitation**

#### **1. Understanding the Access Control Flaw**

The application uses the `id` parameter in the URL to determine which user’s account data is displayed. Instead of verifying access based on the authenticated session, it trusts the user-supplied `id` parameter, allowing unauthorized access to other accounts by modifying this value.

#### **2. Exploiting the Vulnerability**

**Step 1: Log In and Inspect the URL**

1. Log in using **wiener:peter**.
2. Navigate to **your account page** and note the URL structure:
   * **Example:** `/my-account?id=wiener`.
3. Observe the API key displayed on the page.

**Step 2: Send the Request to Burp Repeater**

1. In **Burp Suite**, locate the `GET /my-account?id=wiener` request in the **HTTP history**.
2. Send the request to **Repeater** for further testing.

**Step 3: Manipulate the `id` Parameter**

1. Modify the `id` parameter in the request:
   * Change `id=wiener` to `id=carlos`.
   * **Example:** `GET /my-account?id=carlos`.
2. Send the modified request.

**Step 4: Extract the API Key**

1. Review the response in Burp Suite.
2. Navigate to the **Render** tab and confirm that **Carlos’s account page is now visible**.
3. Copy **Carlos’s API key** from the response.

**Step 5: Submit the API Key to Complete the Lab**

1. Submit Carlos’s API key to **solve the lab**.

***

### **Why This Works**

* The application **incorrectly relies on user input (`id` parameter)** to determine access rights.
* Instead of verifying access based on **session authentication**, it allows any user to supply **any ID**, leading to unauthorized data exposure.
* **Proper access control should be enforced server-side** to validate whether the requesting user has permission to access the specified resource.

***

### **Key Takeaways**

#### **1. The Root Cause of the Vulnerability**

* The server **fails to enforce proper access control** and trusts user-supplied parameters.
* A valid session does not necessarily equate to **proper authorization**.

#### **2. The Impact of the Vulnerability**

* An attacker can **view sensitive information** from another user’s account.
* Similar flaws could allow **modification or deletion of another user’s data**.

#### **3. Best Practices for Preventing ID-Based Access Control Flaws**

* **Enforce server-side access control:** Always validate user permissions against session authentication rather than user-controlled parameters.
* **Use indirect object references:** Instead of exposing sequential `id` values, use UUIDs or session-bound tokens.
* **Implement role-based access controls (RBAC):** Restrict access based on user roles and permissions.
* **Monitor and log unauthorized access attempts:** Detect and block suspicious activities, such as repeated requests with different `id` values.

***

### **Security Implications in Real-World Applications**

#### **Example: BFLA in a Web Application**

During a **penetration test**, a similar **Broken Function Level Authorization (BFLA)** vulnerability was identified:

1. A user (`John`) had access to an API for managing stored data (`/vault/delete`).
2. The tester attempted a **privilege escalation attack** by replacing John’s session cookie with another user's session (`Test`).
3. The request was **wrongly accepted**, allowing unauthorized deletion of another user’s vault entry.
4. **Impact:** Any authenticated user could perform **admin-level actions**, leading to data manipulation risks.

***

### **Final Thoughts**

This lab highlights **how weak access control mechanisms can lead to unauthorized data access**. Applications must ensure that sensitive user data is protected **through strong server-side validation and session-based authorization checks**. Identifying and mitigating these flaws is critical for securing web applications and preventing **data leaks, privilege escalation, and unauthorized modifications**.
