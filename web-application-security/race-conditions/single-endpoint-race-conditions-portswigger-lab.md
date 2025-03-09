# Single-Endpoint Race Conditions (PortSwigger Lab)

## **Lab: Single-Endpoint Race Conditions**

**Lab URL:**\
[Lab: Single-endpoint race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-single-endpoint)

***

### **High-Level Summary**

#### **Objective**

Exploit a race condition in the email change functionality to claim the email address `carlos@ginandjuice.shop`, which has an admin invitation. Once the email change is successful, access the **Admin Panel** and delete the user `carlos`.

#### **Key Techniques & Concepts**

* **Race Conditions:** Exploiting simultaneous requests to manipulate account properties.
* **Parallel Requests:** Sending multiple concurrent requests to force unintended behavior.
* **Session Overwrite:** Intercepting and modifying authentication parameters to take control of an account.

#### **Why This is Vulnerable**

* The application does not properly **lock the account update process**, allowing multiple simultaneous requests to modify the **pending email change state**.
* This allows an attacker to **send multiple email change requests in parallel**, forcing the system to **associate the wrong confirmation token with a new email**.
* By winning the race, the attacker can take ownership of [**carlos@ginandjuice.shop**](mailto:carlos@ginandjuice.shop), an account that has an **admin invite pending**.

***

### **PortSwigger Solution**

#### **Step 1: Identify the Vulnerability**

1.  Log in using the provided credentials:

    ```
    Username: wiener  
    Password: peter  
    ```
2.  Navigate to **My Account** and attempt to change the email to:

    ```
    test@exploit-<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net
    ```
3. Capture the **POST /my-account/change-email** request in **Burp Suite**.
4. Observe that the system sends a **confirmation email** containing a unique token.
5. If a new email change request is submitted, the **previous confirmation token becomes invalid**, indicating a **single-session state management issue**.

***

#### **Step 2: Test for a Race Condition**

1. Remove the applied email change request.
2. Send the **POST /my-account/change-email** request to **Repeater**.
3. Duplicate the request **9 times** for a total of 10 requests.
4. Modify each request with unique email addresses:
   * `test1@exploit-...`
   * `test2@exploit-...`
   * ... up to `test10@exploit-...`
5. Send the requests **one at a time** and monitor the responses.
6. If only one email change is accepted, this confirms a **single pending state** vulnerability.

***

#### **Step 3: Exploit the Race Condition**

1. Create a new **Repeater Tab Group** in **Burp Suite**.
2. Modify two requests:
   * **Request 1:** Change email to `test@exploit-...`
   * **Request 2:** Change email to `carlos@ginandjuice.shop`
3. Select both requests and send them **in parallel**.
4. Refresh your email inbox. If the **confirmation email** sent to `test@exploit-...` contains the token for `carlos@ginandjuice.shop`, proceed with confirming it.
5. Click the confirmation link.
6. Refresh the **My Account** page and verify that your email has changed to `carlos@ginandjuice.shop`.

***

#### **Step 4: Verify Admin Privileges**

1. Once logged in with the `carlos@ginandjuice.shop` email:
2. Navigate to `/admin` and verify that you now have **admin access**.
3. Locate the user `carlos` and delete the account to complete the lab.

✅ **Lab completed successfully.**

***

### **Instructor Solution**

#### **Step 1: Set Up the Attack**

* **Log in** and navigate to **My Account**.
* Attempt to **change the email address** and capture the `POST /my-account/change-email` request.
* Observe the **confirmation token mechanism**.

#### **Step 2: Identify the Flaw**

* If the confirmation token **changes every time a new request is made**, this confirms a **single-session state for pending email updates**.
* This indicates the **possibility of a race condition**.

#### **Step 3: Exploit the Race Condition**

* Set up **two email change requests**:
  * One with `test@exploit-...`
  * One with `carlos@ginandjuice.shop`
* Send them **in parallel** using **Burp Suite Intruder** or **Turbo Intruder**.
* Check the inbox and confirm the **incorrect token association**.
* Click the confirmation link to take ownership of [**carlos@ginandjuice.shop**](mailto:carlos@ginandjuice.shop).

#### **Step 4: Gain Admin Access**

* After confirming the new email, refresh the page and **access the Admin Panel**.
* Delete `carlos` to complete the lab.

***

### **Explanation of the Race Condition**

#### **Why This Worked**

* The system does not enforce **transaction locking**, allowing multiple email change requests to **overwrite each other**.
* The **last request processed wins**, meaning if multiple requests are sent, **one might incorrectly associate a confirmation token** with the **wrong email**.

#### **Indicators of Vulnerability**

* **Single Pending State for Email Changes:** Only one email change request is active at a time.
* **Confirmation Token Reuse:** Multiple simultaneous requests result in **misassigned confirmation tokens**.
* **Lack of Concurrency Control:** No locking mechanism prevents **multiple email changes in parallel**.

***

### **Recommended Fixes and Mitigation Strategies**

#### **1. Implement Transaction Locking**

* Use **atomic database transactions** to ensure only **one email change request** is processed at a time.

```sql
BEGIN TRANSACTION;
UPDATE users SET pending_email = ? WHERE id = ?;
COMMIT;
```

***

#### **2. Use Unique Confirmation Tokens for Each Request**

* Generate **unique, time-sensitive confirmation tokens** per request and **invalidate old tokens** when a new request is made.

```python
def generate_token(email, user_id):
    return hashlib.sha256((email + str(user_id) + secret_key).encode()).hexdigest()
```

***

#### **3. Enforce Rate Limiting**

* Implement **rate limiting** to prevent **high-speed automated requests** from exploiting the race condition.

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
server {
    location /my-account/change-email {
        limit_req zone=one burst=5 nodelay;
    }
}
```

***

#### **4. Require Manual Confirmation for Critical Actions**

* Implement **multi-step verification**, requiring users to **enter their current password** before changing their email.

```javascript
if (user.submitted_password !== user.current_password) {
    return { "error": "Password required for email change" };
}
```

***

### **Real-World Examples of Race Conditions**

#### **CVE-2020-9283 - GitHub Enterprise Race Condition**

* A race condition in GitHub Enterprise allowed users to **overwrite repository permissions**, granting unauthorized access.
* Attackers exploited a **simultaneous request flaw** to manipulate ownership records.
* [Reference: CVE-2020-9283](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-9283)

***

### **Key Takeaways**

* **Race conditions** occur when **multiple concurrent requests** modify **session-sensitive data**.
* **Parallel request flooding** can cause **unexpected state changes**, such as **email overwrites**.
* **Fixing race conditions requires** transactional locking, unique request tracking, and rate limiting.

***

#### **Final Thoughts**

* **Identify race-prone endpoints** where state updates rely on **sequential processing**.
* **Test with parallel requests** to observe **unexpected session overwrites**.
* **Secure critical account functions** by implementing **locking, rate limiting, and multi-step verification**.
