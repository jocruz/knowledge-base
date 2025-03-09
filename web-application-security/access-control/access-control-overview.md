# Access Control Overview

## **Access Control Overview**

**What is Access Control?**

Access control, also known as **authorization**, determines what users are permitted to do within a system or application. While authentication verifies a user’s identity, access control ensures users can only perform **authorized actions** and access the appropriate resources based on their roles and permissions.

Effective access control mechanisms are essential for preventing **unauthorized data access**, **privilege escalation**, and **security breaches** in modern applications.

***

### **Types of Access Controls**

#### **1. Horizontal Access Control**

* Restricts users to **their own data or objects**.
* **Example:** A user should only be able to view **their own** bank account details and not those of others.

#### **2. Vertical Access Control**

* Restricts users based on **privilege levels** (e.g., user vs. admin).
* **Example:** A **regular user** cannot modify system-wide settings or access administrative functions.

#### **3. Context-Dependent Access Control**

* Restricts access based on **application state** or external factors.
* **Example:** A **customer cannot purchase** an item before its official release date.

***

### **Common Access Control Vulnerabilities**

#### **1. Insecure Direct Object Reference (IDOR)**

* Occurs when an attacker manipulates an identifier in a request (e.g., a user ID in a URL) to gain unauthorized access to another user's data.
* **Example:** Changing `/profile?user_id=1023` to `/profile?user_id=1024` to view another user’s profile.

#### **2. Broken Object Level Authorization (BOLA)**

* Common in APIs where **authorization checks are missing** or insufficient.
* **Example:** A mobile app request for `/orders?customer_id=123` returns all customer orders, but an attacker modifies it to `/orders?customer_id=124` to access someone else's data.

#### **3. Broken Function Level Authorization (BFLA)**

* Occurs when **privileged actions** are accessible to unauthorized users.
* **Example:** A regular user finds they can **delete another user’s account** by submitting a direct request to `/admin/delete_user`.

#### **4. Forceful Browsing**

* A user accesses **restricted pages** by directly navigating to a URL without proper authorization checks.
* **Example:** A non-admin user enters `/admin-dashboard` in their browser and gains access to the admin panel.

***

### **Best Practices for Securing Access Control**

#### **1. Enforce Proper Authorization Checks**

* Validate **user permissions at every endpoint** before granting access.
* Implement **server-side access controls** (client-side restrictions can be bypassed).

#### **2. Secure Object-Level Access (Prevent IDOR & BOLA)**

* Use **indirect references** (e.g., UUIDs instead of sequential IDs).
* Ensure **every request verifies ownership** before returning data.

#### **3. Protect Against Forceful Browsing**

* Restrict access to sensitive areas with **role-based authentication**.
* Ensure **unauthorized requests return HTTP 403 (Forbidden)** instead of leaking information.

#### **4. Limit Privilege Escalation Risks (Prevent BFLA)**

* Implement **role-based access control (RBAC)** with strict privilege levels.
* Regularly audit **user permissions and access logs**.

***

### **Why Access Control Matters**

Access control is a **fundamental security principle** that prevents unauthorized access, protects user data, and ensures applications operate securely. Without proper access controls, attackers can **escalate privileges, manipulate API requests, and exploit authentication flaws** to compromise sensitive data.

By implementing **robust access control mechanisms**, organizations can significantly reduce security risks and ensure that **users only have access to the resources and functionality they are authorized to use**.
