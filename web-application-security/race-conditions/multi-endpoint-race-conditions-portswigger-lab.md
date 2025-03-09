# Multi-Endpoint Race Conditions (PortSwigger Lab)

## **Lab: Multi-Endpoint Race Conditions**

**Lab URL:**\
[Web Security Academy](https://portswigger.net/web-security/race-conditions/lab-multi-endpoint)

***

### **High-Level Summary**

#### **Objective**

Exploit a race condition in the purchasing process to buy the **Lightweight L33t Leather Jacket** at a significantly reduced price.

#### **Key Techniques & Concepts**

* **Race Conditions:** Exploiting simultaneous requests to manipulate transaction validation.
* **Parallel Requests:** Sending concurrent requests to bypass price calculations.
* **Multi-Endpoint Exploitation:** Targeting separate **cart** and **checkout** requests to interfere with the purchase process.

#### **Why This is Vulnerable**

* The application does not **synchronize** transaction processing properly.
* A race condition occurs when **multiple requests** are sent simultaneously, causing the system to process **both adding the item and checkout before fully validating the purchase**.
* This allows an attacker to **checkout an expensive item while appearing to purchase a cheaper item**.

***

### **PortSwigger Solution**

#### **Step 1: Identify the Vulnerability**

1.  Log in using the provided credentials:

    ```
    Username: wiener  
    Password: peter  
    ```
2. Add a **gift card** to the cart and proceed to checkout.
3. Capture the following key requests in **Burp Suite**:
   * `POST /cart` – Adds an item to the cart.
   * `POST /cart/checkout` – Confirms and processes the purchase.
4. Examine how the store processes the purchase, particularly **how the cart updates when checkout occurs**.

***

#### **Step 2: Test for a Race Condition**

1. Remove all items from the cart.
2. Add a **cheap item (gift card)** to the cart.
3. Send the `POST /cart` and `POST /cart/checkout` requests to **Burp Repeater**.
4. **Modify the Requests:**
   * Change the `productId` parameter from `2` (gift card) to `1` (leather jacket).
5. Send the requests sequentially and observe:
   * If the purchase fails due to insufficient funds, a **race condition may exist**.

***

#### **Step 3: Exploit the Race Condition**

1. **Set Up the Attack**
   * Remove all cart items and add a **gift card** back.
   * Capture both `POST /cart` and `POST /cart/checkout` requests.
2. **Create a Parallel Attack Group**
   * Send **both requests in parallel** using Burp’s **Parallel Request Mode**.
   * Modify `productId` in one request to **1 (leather jacket)** while keeping the other as the **gift card**.
3. **Execute the Attack**
   * Send multiple parallel requests until one succeeds.
   * If done correctly, the store processes the **leather jacket purchase at the discounted price**.

✅ **If successful, the leather jacket is purchased at a significantly reduced price, completing the lab.**

***

### **Instructor Solution**

#### **Step 1: Understanding the Checkout Process**

* Log in and add a **gift card** to the cart.
* Observe that when submitting `POST /cart/checkout`, the store processes the **cart contents at the time of checkout**.
* A race condition exists if **the cart can be modified mid-transaction**.

#### **Step 2: Preparing the Attack**

* Capture the following requests in **Burp Suite**:
  * `POST /cart`
  * `POST /cart/checkout`
* Modify the `productId` value in the `/cart` request:
  * **Original request:** Gift card (`productId=2`)
  * **Modified request:** Leather jacket (`productId=1`)

#### **Step 3: Executing the Attack**

* Set up Burp’s **Parallel Request Mode**.
* Send both the modified `/cart` and `/cart/checkout` requests simultaneously.
* If successful, the transaction completes before the system fully verifies the **actual cart contents**.

✅ **Key Takeaways from the Instructor:**

* The store **fails to lock transactions**, allowing modifications mid-checkout.
* **Parallel request execution** forces unintended order processing.

***

### **Explanation of the Race Condition**

#### **Why This Worked**

* The server validates the cart **before processing checkout**.
* **Parallel requests** cause:
  * One request to **add the leather jacket**.
  * Another request to **checkout** before validation updates.

#### **Indicators of Vulnerability**

* **Separate endpoints for cart updates and checkout processing.**
* **No transaction locking** between the `/cart` and `/cart/checkout` processes.
* **Delayed state updates** allow multiple requests to process conflicting actions.

***

### **Mitigation Strategies**

#### **1. Implement Transaction Locking**

* Use **atomic database transactions** to prevent simultaneous updates.

```sql
BEGIN TRANSACTION;
UPDATE cart SET item = ? WHERE user_id = ?;
COMMIT;
```

***

#### **2. Ensure Atomicity in Checkout**

* Checkout validation should happen **at the moment of transaction processing**, not before.

```python
def checkout(user_id):
    cart = get_cart(user_id)
    validate_funds(user_id, cart.total)
    process_order(user_id, cart)
```

***

#### **3. Implement Rate Limiting**

* Limit how frequently checkout requests can be made in a short time frame.

```nginx
limit_req_zone $binary_remote_addr zone=checkout:10m rate=1r/s;
```

***

#### **4. Revalidate Cart at Checkout**

* **Verify cart contents just before processing the order**, preventing mid-transaction modification.

```javascript
if (!cart.matches(previous_cart_state)) {
    return { "error": "Cart contents changed. Please retry checkout." };
}
```

***

### **Real-World Examples of Race Conditions**

#### **CVE-2019-15078 - Stripe API Race Condition**

* A vulnerability in Stripe allowed users to **double-spend coupons** by sending **simultaneous API requests**.
* Attackers exploited this by **applying discounts multiple times** before the system updated transaction states.
* [Reference: CVE-2019-15078](https://nvd.nist.gov/vuln/detail/CVE-2019-15078)

***

### **Key Takeaways**

* **Multi-endpoint race conditions** occur when separate API calls modify shared states.
* **Parallel execution of checkout and cart updates** can force price mismatches.
* **Preventing race conditions** requires proper transaction management, rate limiting, and atomic validation.

***

#### **Final Thoughts**

* **Identify separate API actions** that interact with shared state (e.g., cart and checkout).
* **Use Burp Suite’s parallel request execution** to simulate race conditions.
* **Secure transactional logic** by enforcing consistency and validation at the moment of execution.

By applying these security measu
