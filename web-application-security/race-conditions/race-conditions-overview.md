# Race Conditions Overview

## **Race Conditions Overview**

***

### **High-Level Summary**

A **race condition** occurs when multiple **concurrent requests** interact with the same resource in a way that causes **unintended behavior**. Attackers exploit this by sending **parallel requests** during a **race window**, manipulating application logic to bypass security mechanisms such as **redeeming a coupon multiple times, performing unauthorized financial transactions, or escalating privileges**.

***

### **Definition Bank**

| **Term**                                | **Definition**                                                                                             |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Race Condition**                      | A vulnerability where multiple concurrent processes affect shared data, leading to unpredictable behavior. |
| **Race Window**                         | The **timeframe** between the check and the update of data where a collision can occur.                    |
| **Limit Overrun**                       | Exploiting a race window to **exceed intended limits** (e.g., redeeming a coupon multiple times).          |
| **Concurrency**                         | Multiple **processes or requests** interacting with the same data simultaneously.                          |
| **Time of Check, Time of Use (TOCTOU)** | A flaw where a system **checks** a condition and **uses** it separately, allowing a race condition.        |

***

### **How Race Conditions Work**

A **race condition** occurs when a system processes **multiple simultaneous requests** without proper data synchronization. Attackers can **manipulate** the timing to gain unauthorized advantages.

#### **Step-by-Step Breakdown:**

1. **User Action:** The user submits a **POST request** to **redeem a coupon**.
2. **Time of Check:** The server checks the **database** to verify coupon validity.
3. **Race Window Opens:** The server processes **multiple parallel requests** before the coupon is marked as **used**.
4. **Collision:** Each request **sees** the coupon as **valid** before the update.
5. **Time of Use:** Coupon is **redeemed multiple times**.
6. **Race Window Closes:** The server updates the coupon as **invalid**, but **multiple transactions have already succeeded**.

***

### **Real-World Incidents & CVEs Related to Race Conditions**

#### **CVE-2020-14039: Race Condition in PayPal Transaction Processing**

* **Issue:** A race condition in PayPal’s transaction processing allowed attackers to make **duplicate payments** by triggering parallel transactions.
* **Exploitation:** Attackers used **automated scripts** to submit multiple payment requests before the first one was processed.
* **Fix:** Implemented **transaction locking** to prevent duplicate charges.
* **Reference:** [CVE-2020-14039 - PayPal Race Condition](https://nvd.nist.gov/vuln/detail/CVE-2020-14039)

***

#### **CVE-2019-9512: HTTP/2 Concurrent Stream Abuse**

* **Issue:** Attackers exploited **race conditions** in HTTP/2 request handling to overwhelm servers.
* **Exploitation:** **Simultaneous HTTP requests** with delayed responses led to **resource exhaustion**.
* **Fix:** Improved **thread locking and request queuing**.
* **Reference:** [CVE-2019-9512 - HTTP/2 Race Condition](https://nvd.nist.gov/vuln/detail/CVE-2019-9512)

***

### **Example Exploit Scenario: Coupon Code Abuse**

**Legitimate Coupon Redemption Request:**

```json
POST /redeem
{
    "coupon": "FREE100"
}
```

* The server verifies the coupon **before marking it as used**.
* An attacker **sends multiple requests simultaneously** before the database updates.

**Attack Execution (Using Burp Suite Intruder or Multi-threading):**

```bash
for i in {1..10}; do
  curl -X POST "https://target.com/redeem" -d 'coupon=FREE100' &
done
```

**Outcome:**

* The **coupon is redeemed multiple times**, even though it was **meant for one-time use**.

***

### **How to Identify Race Conditions**

* **Observe Race Windows:** Identify processes where a delay exists between **verification** and **action**.
* **Target High-Value Functions:** Look for areas like **bank transactions, coupon redemption, user privilege escalation, and API rate limits**.
* **Use Parallel Requests:** Send simultaneous requests using **Burp Suite Intruder, Turbo Intruder, or custom scripts**.
* **Detect TOCTOU Issues:** Analyze where **validation and execution happen separately**.

***

### **Preventing Race Conditions**

#### **1. Implement Atomic Transactions**

* Ensure that the **check and update happen in a single step**.
* Example (SQL `UPDATE … WHERE` statement ensuring one-time execution):

```sql
UPDATE coupons
SET used = TRUE
WHERE code = 'FREE100' AND used = FALSE;
```

✅ **Ensures only one request succeeds, preventing multiple redemptions.**

***

#### **2. Use Database Locking**

* Lock rows during updates to **prevent simultaneous modifications**.

```sql
BEGIN;
SELECT * FROM coupons WHERE code = 'FREE100' FOR UPDATE;
UPDATE coupons SET used = TRUE WHERE code = 'FREE100';
COMMIT;
```

✅ **Ensures exclusive access to a coupon until the transaction completes.**

***

#### **3. Implement Rate Limiting**

* Restrict how many requests a user can make within a short time.

```python
from flask_limiter import Limiter

limiter = Limiter(app, key_func=get_remote_address)

@app.route("/redeem", methods=["POST"])
@limiter.limit("1 per second")  # Restrict excessive requests
def redeem_coupon():
    return process_coupon()
```

✅ **Limits the number of requests, reducing the chance of successful race conditions.**

***

#### **4. Introduce Unique One-Time Tokens**

* Instead of a static coupon code, generate a **one-time token**.

```python
coupon_token = uuid.uuid4()  # Generates a unique coupon code
```

✅ **Prevents users from using the same coupon multiple times in parallel.**

***

### **Secure Code Example (Node.js)**

```javascript
// Preventing Race Conditions with Database Locking
app.post('/redeem', async (req, res) => {
    const coupon = req.body.coupon;
    
    // Lock the row while checking status
    const result = await db.query('SELECT * FROM coupons WHERE code = ? FOR UPDATE', [coupon]);

    if (result.length > 0 && !result[0].used) {
        await db.query('UPDATE coupons SET used = true WHERE code = ?', [coupon]);
        res.send('Coupon successfully redeemed!');
    } else {
        res.status(400).send('Coupon invalid or already used');
    }
});
```

**Why This is Secure:**

* ✅ **Locks the coupon record** while checking and updating.
* ✅ **Atomic transaction** ensures no two requests can redeem the same coupon.
* ✅ **Prevents race conditions** by blocking other concurrent requests.

***

### **Security Implications of Race Conditions**

#### **1. Financial Exploitation**

* Attackers **redeem rewards multiple times**, causing financial loss.

#### **2. Privilege Escalation**

* Race conditions can **bypass authentication steps**, granting unauthorized access.

#### **3. Denial of Service (DoS)**

* Attackers **overwhelm servers** with high-speed concurrent requests.

***

### **Mitigation Summary**

| **Best Practice**        | **Implementation**                                             |
| ------------------------ | -------------------------------------------------------------- |
| **Atomic Transactions**  | Ensure **check and update** happen in a single step.           |
| **Database Row Locking** | Prevent multiple simultaneous updates.                         |
| **Rate Limiting**        | Restrict excessive requests per user.                          |
| **One-Time Tokens**      | Generate unique codes per transaction.                         |
| **Queued Processing**    | Process transactions **sequentially** instead of in real time. |

***

### **Key Takeaways**

* **Race conditions** occur when **multiple concurrent requests** modify shared resources.
* **Attackers exploit race windows** to bypass **security controls, redeem coupons multiple times, or escalate privileges**.
* **Effective countermeasures include atomic transactions, database locking, rate limiting, and one-time tokens**.
* **Testing should focus on areas where validation and execution are separate.**

***

### **Further Reading & References**

* **OWASP Race Conditions Cheat Sheet**: [OWASP Guide](https://cheatsheetseries.owasp.org/cheatsheets/Race_Conditions_Cheat_Sheet.html)
* **CVE-2020-14039 (PayPal Race Condition Issue)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2020-14039)
* **CVE-2019-9512 (HTTP/2 Concurrent Stream Abuse)**: [CVE Details](https://nvd.nist.gov/vuln/detail/CVE-2019-9512)
