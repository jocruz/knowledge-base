# IDOR (Insecure Direct Object Reference) Overview

## **IDOR (Insecure Direct Object Reference) Overview**

**What is IDOR?**

**Insecure Direct Object Reference (IDOR)** occurs when an application **allows user-supplied input to directly reference resources (e.g., database entries, files, or accounts) without proper authorization checks**. This vulnerability enables attackers to manipulate object references, such as changing an `id` value in a request, to access unauthorized data.

IDOR is a subset of **Broken Object-Level Authorization (BOLA)**, which describes the broader failure of enforcing authorization at the object level. It also relates to **Broken Access Control (BAC)** and **Broken Function-Level Authorization (BFLA)** when authorization flaws extend to actions rather than objects.

***

### **Key Takeaways**

#### **1. How IDOR Works**

* Applications expose **object references** (e.g., user IDs, document numbers) that can be manipulated.
* Attackers modify these references to access **another user’s account or data**.
* Commonly found in:
  * **URLs**: `/profile?id=123 → id=124` (viewing another user's profile).
  * **APIs**: `GET /api/orders/5678` (retrieving another customer’s order).
  * **File Access**: `GET /files/contract-123.pdf → contract-124.pdf`.

#### **2. Related Access Control Issues**

* **BOLA (Broken Object-Level Authorization):** IDOR is a subset of BOLA, which occurs when applications fail to verify user access to specific objects.
* **BFLA (Broken Function-Level Authorization):** Allows unauthorized users to perform actions beyond their permissions (e.g., user modifying admin settings).
* **BAC (Broken Access Control):** A broad category that includes IDOR, BOLA, and BFLA, referring to any failure in enforcing proper user permissions.

***

### **Common IDOR Vulnerabilities**

#### **1. URL Parameter Manipulation**

* **Example:** Changing `id=123` to `id=124` in `/user/profile?id=124` to access another user’s data.

#### **2. API Authorization Flaws (BOLA)**

* **Example:** A mobile banking app retrieves transactions using `GET /api/transactions?account=5678`. If the API does not validate ownership, an attacker can access another user’s transactions by changing `account=5678` to `account=5679`.

#### **3. File and Resource Access Issues**

* **Example:** An invoicing system allows downloading invoices via `GET /invoices/123.pdf`. If access control is missing, changing `123.pdf` to `124.pdf` might expose another customer’s invoice.

#### **4. JSON and XML Data Exposure**

*   **Example:** A request to fetch user data:

    ```json
    {
        "user_id": "123"
    }
    ```

    If the application does not verify ownership, sending `user_id: "124"` might return another user's data.

***

### **Best Practices for Preventing IDOR**

#### **1. Enforce Server-Side Authorization Checks**

* Validate **every request** to ensure the requesting user has permission to access the specified object.
* Do not rely on client-side controls (e.g., hidden fields or JavaScript checks).

#### **2. Implement Indirect Object References**

* Instead of exposing sequential IDs, use **UUIDs or hashed references**:
  * **Example:** Instead of `GET /profile?id=123`, use `GET /profile?id=ec9b88a3f6a14f1e9b7c7d`.

#### **3. Restrict API and URL-Based Access**

* Ensure **authentication and authorization checks** are in place for all API endpoints.
* Use **token-based access controls** to prevent unauthorized data retrieval.

#### **4. Log and Monitor Requests**

* Implement **audit logs** to detect unusual object access patterns.
* Alert security teams on **multiple failed object access attempts** (e.g., rapid changes to `id` values in requests).

***

### **Why IDOR Matters**

IDOR vulnerabilities are among the **most common and impactful access control flaws**, leading to:

* **Data breaches**: Exposure of personal information, financial records, or sensitive files.
* **Account takeover risks**: Unauthorized access to user profiles or settings.
* **Regulatory non-compliance**: Violations of **GDPR, HIPAA, or PCI-DSS** due to data exposure.

By implementing **proper authorization controls, indirect object references, and server-side validation**, organizations can effectively mitigate IDOR and related access control vulnerabilities.
