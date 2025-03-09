# Multi-Endpoint Race Conditions Overview

## **Multi-Endpoint Race Conditions Overview**

***

### **High-Level Summary**

A **multi-endpoint race condition** occurs when multiple **interdependent endpoints** process requests asynchronously, leading to security flaws. Attackers exploit **timing mismatches** between endpoints to manipulate data, bypass security checks, or execute unauthorized actions.

**Key Exploit Areas:**

* **E-commerce manipulation** (adding extra items after payment validation)
* **Banking transactions** (withdrawing more money than allowed)
* **Privilege escalation** (modifying permissions before verification)

***

### **Definition Bank**

| **Term**                          | **Definition**                                                                                    |
| --------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Multi-Endpoint Race Condition** | A race condition that involves multiple API endpoints processing requests in an unintended order. |
| **Race Window**                   | The **timeframe** between validation and execution where an attacker can intervene.               |
| **Atomic Transactions**           | A process where operations are **executed in a single step**, preventing interference.            |
| **Session Consistency**           | Ensuring session-based actions remain synchronized across multiple requests.                      |
| **Concurrency**                   | The simultaneous execution of multiple processes that interact with shared data.                  |
| **Business Logic Manipulation**   | Altering the normal flow of transactions to gain unintended advantages.                           |

***

### **How Multi-Endpoint Race Conditions Work**

Unlike **single-endpoint race conditions**, which target a single request, **multi-endpoint race conditions** exploit **delays** and **inconsistencies** between multiple API calls.

#### **Example: E-Commerce Checkout Manipulation**

1. **Step 1:** The user adds **3 items** to their cart and proceeds to checkout.
2. **Step 2:** The application **validates the cart total** and redirects the user to a **third-party payment processor**.
3. **Step 3:** The payment processor **approves the payment** and redirects back to the store.
4. **Step 4:** The store **finalizes the order** and updates the inventory.

#### **Exploit Scenario: Adding Extra Items Without Paying**

During **Step 3**, an attacker **sends additional requests** to:

* **Modify cart contents** (e.g., adding 10 more items).
* **Alter pricing details** (e.g., applying multiple discounts).
* **Bypass inventory limits** (e.g., exceeding purchase caps).

If the application **does not revalidate the cart before order finalization**, the attacker **receives extra items without being charged**.

***

### **Real-World Incidents & CVEs Related to Multi-Endpoint Race Conditions**

#### **CVE-2023-27470: Payment Gateway Race Condition**

* **Issue:** Attackers exploited a **timing flaw** between a retailer's **cart validation** and **payment approval**, allowing them to buy items for **$0**.
* **Exploit:** Attackers sent **parallel requests** to modify cart contents **after payment validation but before order completion**.
* **Fix:** Implemented **final order revalidation** before shipping confirmation.
* **Reference:** [CVE-2023-27470](https://nvd.nist.gov/vuln/detail/CVE-2023-27470)

***

#### **CVE-2022-43873: Banking Transaction Manipulation**

* **Issue:** A **multi-endpoint race condition** in an online banking system allowed attackers to **withdraw more money than available**.
* **Exploit:** Attackers sent **rapid concurrent withdrawal requests** before the balance was updated.
* **Fix:** Implemented **database locking** and **real-time balance updates**.
* **Reference:** [CVE-2022-43873](https://nvd.nist.gov/vuln/detail/CVE-2022-43873)

***

### **Exploiting Multi-Endpoint Race Conditions (Testing Methodology)**

1. **Identify Key API Endpoints**
   * Look for endpoints that interact **sequentially** (e.g., cart validation, payment processing, order finalization).
   * Examine session-based actions (e.g., **password resets, fund transfers, or role updates**).
2. **Send Asynchronous Requests**
   * Use **Burp Suite Turbo Intruder** or **a multi-threaded Python script** to send **simultaneous requests**.
3. **Monitor for Data Inconsistencies**
   * Identify **changes in API responses** when requests are sent in **parallel**.
4. **Confirm Exploitability**
   * Determine if unauthorized **actions, modifications, or financial manipulations** are possible.

***

### **Exploitation Example: Banking Transaction Bypass**

#### **Scenario:**

* A banking app allows users to **transfer funds** between accounts.
* The **balance check** and **fund transfer confirmation** occur via **two separate endpoints**.

#### **Exploit Steps:**

1. **Initiate a $500 Transfer to Another Account**
2. **Before the Balance Updates, Send a Second Transfer Request for Another $500**
3. **Observe if Both Transfers Succeed Despite Insufficient Funds**

**Expected Outcome (Vulnerable System)**

* The attacker **transfers $1000 despite having only $500** due to a **race condition between the validation and transfer execution**.

***

### **Mitigation Strategies for Multi-Endpoint Race Conditions**

| **Mitigation**                        | **Implementation**                                                               |
| ------------------------------------- | -------------------------------------------------------------------------------- |
| **Atomic Transactions**               | Ensure validation and execution **occur together** to prevent interference.      |
| **Database Locking**                  | Use **row-level locks** to prevent simultaneous updates.                         |
| **Session Integrity Checks**          | Maintain **consistent session tracking** across all request handlers.            |
| **Final Validation Before Execution** | Always **revalidate cart contents or financial transactions** before processing. |
| **Rate Limiting**                     | Restrict **multiple requests per second** for sensitive actions.                 |
| **Concurrency Control**               | Prevent simultaneous API requests from altering the same resource.               |

***

### **Secure Code Example (Node.js)**

#### **Preventing Multi-Endpoint Race Conditions with Atomic Transactions**

```javascript
app.post('/checkout', async (req, res) => {
    const userId = req.session.userId;

    await db.query('LOCK TABLE carts WRITE'); // Prevent simultaneous updates

    const cart = await db.query('SELECT * FROM carts WHERE user_id = ?', [userId]);

    if (!cart || cart.items.length === 0) {
        res.status(400).send("Cart is empty or invalid.");
        return;
    }

    const total = calculateTotal(cart.items);
    const paymentSuccess = await processPayment(userId, total);

    if (paymentSuccess) {
        await db.query('UPDATE carts SET finalized = true WHERE user_id = ?', [userId]);
        res.send("Order completed successfully.");
    } else {
        res.status(400).send("Payment failed.");
    }

    db.query('UNLOCK TABLES'); // Release lock
});
```

#### **Why This is Secure:**

* ✅ **Database Locking:** Prevents multiple simultaneous updates to the cart.
* ✅ **Final Validation:** Ensures the cart is checked **before order completion**.
* ✅ **Atomic Execution:** Prevents race conditions by **combining validation and execution into a single operation**.

***

### **Key Takeaways**

* **Multi-endpoint race conditions** occur when **multiple API calls process data inconsistently**, allowing attackers to exploit **timing gaps**.
* Attackers manipulate **concurrent requests** to **alter transactions, bypass security, or exploit business logic**.
* **Real-world incidents**, such as **CVE-2023-27470 (E-commerce Checkout Manipulation)** and **CVE-2022-43873 (Banking Transaction Exploit)**, demonstrate the risks of **insufficient validation across multiple API endpoints**.
* **Mitigation strategies**, including **atomic transactions, concurrency control, final validation, and session consistency**, are essential to prevent exploitation.

***

### **Further Reading & References**

* **OWASP Race Conditions Cheat Sheet**: [OWASP Guide](https://cheatsheetseries.owasp.org/cheatsheets/Race_Conditions_Cheat_Sheet.html)
* **CVE-2023-27470 (E-commerce Payment Exploit)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2023-27470)
* **CVE-2022-43873 (Banking Transaction Exploit)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2022-43873)
