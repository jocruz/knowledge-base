# Limit overrun race conditions (PortSwigger Lab)

## **Lab: Limit Overrun Race Conditions**

**Lab URL:**\
[Web Security Academy](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun)

***

### **High-Level Summary**

#### **Objective**

Exploit a race condition in the purchasing process to apply a discount multiple times, allowing the purchase of the **Lightweight L33t Leather Jacket** at a significantly reduced price.

#### **Key Techniques & Concepts**

* **Race Conditions:** Exploiting simultaneous requests to bypass application logic restrictions.
* **Parallel Requests:** Sending multiple requests in rapid succession to trigger unintended behavior.
* **Coupon Exploitation:** Applying the same discount multiple times due to insufficient transaction locking.

#### **Why This is Vulnerable**

* The server does not properly **lock transactions**, allowing multiple concurrent discount applications.
* Requests sent in **parallel** are processed before the system updates the cart’s state.
* As a result, the discount is applied **multiple times**, reducing the item’s price beyond the intended limit.

***

### **PortSwigger Solution**

#### **Step 1: Log in and Identify the Discount Mechanism**

1.  Log in with the provided credentials:

    ```
    Username: wiener  
    Password: peter  
    ```
2. Add a **cheap item** to the cart and apply the `PROMO20` discount code.
3. Capture the **POST /cart/coupon** request in **Burp Suite**.

***

#### **Step 2: Observe How the Server Handles Discounts**

1. Apply the `PROMO20` coupon once.
2.  The server responds with:

    ```json
    {"success": true, "message": "Coupon applied"}
    ```
3.  If the same request is sent again, the server blocks it with:

    ```json
    {"success": false, "message": "Coupon already applied"}
    ```
4. This indicates that the **cart state is stored server-side** and linked to the **session ID**.

***

#### **Step 3: Attempt Sequential Requests**

1. Remove the discount from the cart.
2. Send the **POST /cart/coupon** request **one at a time** using **Burp Suite Repeater**.
3. Observe that only the first request succeeds, while subsequent attempts return `"Coupon already applied"`.

***

#### **Step 4: Execute the Race Condition Attack**

1. **Remove any applied coupons** from the cart.
2. **Send multiple requests in parallel**:
   * Use **Burp Suite’s Intruder** or **Turbo Intruder** to send 20+ identical requests simultaneously.
3. **Expected Outcome:**
   * Some requests succeed in applying the coupon **before** the system updates the session.
   * The discount is **applied multiple times**.

***

#### **Step 5: Validate the Exploit**

1. Add the **Lightweight L33t Leather Jacket** to the cart.
2. Repeat the **parallel request attack** on the **POST /cart/coupon** endpoint.
3. Refresh the cart and verify that the total price is **significantly reduced**.
4. Complete the purchase to finish the lab.

✅ **Lab completed successfully.**

***

### **Instructor Solution**

#### **Step 1: Analyze the Web Application**

* **Log in** and add an item to the cart.
* Apply the `PROMO20` discount and capture the request.

#### **Step 2: Identify the Vulnerability**

* Locate the **POST /cart/coupon** request in **Burp Suite**.
* Observe the cart state update mechanism.
* Confirm that repeated discount applications are blocked **when sent sequentially**.

#### **Step 3: Exploit the Race Condition**

* **Remove the coupon** from the cart.
* Set up **parallel request execution** using Burp Suite:
  * Use **Intruder** with a **cluster bomb attack type**.
  * Use **Turbo Intruder** for high-speed request flooding.
* Send multiple requests **simultaneously**.

#### **Step 4: Verify Multiple Discounts Applied**

* Refresh the cart and confirm multiple discounts are stacked.
* Complete the purchase at a **heavily discounted price**.

***

### **Explanation of the Vulnerability**

#### **Why the Race Condition Works**

* The server does **not synchronize transactions properly**.
* Multiple requests are processed **before** the session state is updated.
* This allows the discount to be **applied multiple times**.

#### **Key Indicators of Vulnerability**

* **State-dependent operations** (like discounts) should have transactional locking.
* The **session-based discount check** is performed **after** the request is processed.
* **No atomic validation** of discount application across simultaneous requests.

***

### **Recommended Fixes and Mitigation Strategies**

#### **1. Implement Transaction Locking**

* Use **database locks** to ensure that only **one** discount application request is processed at a time.

```sql
BEGIN TRANSACTION;
UPDATE cart SET discount_applied = 1 WHERE user_id = 123;
COMMIT;
```

***

#### **2. Use Unique Discount Tracking**

* Store a **list of applied discounts** and check for duplicates **before processing**.

```python
def apply_discount(user_id, coupon_code):
    if coupon_code in applied_discounts[user_id]:
        return {"success": False, "message": "Coupon already applied"}
    applied_discounts[user_id].append(coupon_code)
    return {"success": True, "message": "Coupon applied"}
```

***

#### **3. Enforce Server-Side Rate Limiting**

* **Restrict rapid requests** from the same user or session using rate-limiting controls.

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
server {
    location /cart/coupon {
        limit_req zone=one burst=5 nodelay;
    }
}
```

***

#### **4. Require Explicit User Confirmation**

* Implement **multi-step verification** before applying a discount.
* Require users to **confirm the discount manually** in the UI.

***

### **Real-World Examples of Race Conditions**

#### **CVE-2019-18935 - Telerik UI Race Condition Exploit**

* A race condition in Telerik UI allowed **unauthorized remote code execution**.
* Attackers bypassed security restrictions by **sending concurrent requests**.
* [Reference: CVE-2019-18935](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-18935)

***

### **Key Takeaways**

* **Race conditions** occur when **multiple concurrent requests** manipulate state-dependent operations.
* **Parallel request flooding** can cause **unexpected behavior**, such as **applying the same discount multiple times**.
* **Fixing race conditions requires transactional locking, state validation, and request throttling**.

***

#### **Final Thoughts**

* **Look for endpoints** that involve **state changes** (e.g., cart modifications, account updates).
* **Test for race conditions** by sending **parallel requests** and observing unexpected outcomes.
* **Implement proper synchronization techniques** to prevent unintended behavior.
