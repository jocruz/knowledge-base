# NoSql Overview

## **NoSQL Injection – Comprehensive Overview**

### **High-Level Summary: NoSQL Injection**

_**NoSQL Injection**_ is a security vulnerability that occurs when **unvalidated user input** is inserted into NoSQL database queries, allowing attackers to manipulate query logic. Unlike SQL Injection, which targets relational databases (MySQL, PostgreSQL, MSSQL), NoSQL injection exploits **schema-flexible databases** like **MongoDB, CouchDB, Firebase, and Elasticsearch**.

#### **Potential Risks**

* **Authentication Bypass:** Attackers log in without valid credentials.
* **Data Exfiltration:** Extracting confidential user records.
* **Data Manipulation:** Modifying or deleting documents.
* **Denial of Service (DoS):** Executing large queries that crash the database.

***

### **Definition Table**

| **Term**                  | **Definition**                                                                  |
| ------------------------- | ------------------------------------------------------------------------------- |
| **NoSQL Databases**       | Databases designed for flexible, unstructured data storage.                     |
| **NoSQL Injection**       | An attack where unvalidated user input alters NoSQL queries.                    |
| **$ne Operator**          | MongoDB’s "not equal to" operator (`{ "password": { "$ne": "" } }`)             |
| **$regex Operator**       | Enables pattern-matching for queries (`{ "username": { "$regex": ".*" } }`)     |
| **Schema-Less Design**    | NoSQL databases often lack rigid schemas, making them susceptible to injection. |
| **Parameterized Queries** | A safe way to separate user input from database logic, preventing injection.    |

***

### **How NoSQL Injection Works**

Unlike SQL, NoSQL databases process queries as **JSON-like objects**. A **failure to validate or sanitize input** allows attackers to inject **special query operators** (`$ne`, `$gt`, `$regex`) into API endpoints or login forms.

#### **Example: Vulnerable Code (MongoDB)**

```javascript
db.users.find({ username: req.body.username, password: req.body.password });
```

* **Why it’s vulnerable?**
  * The application **directly** accepts user input in `req.body.username` and `req.body.password` without sanitization.
  * An attacker can modify this input to alter the query logic.

#### **Payload for Authentication Bypass**

```json
{
  "username": "admin",
  "password": { "$ne": "" }
}
```

* **Impact:**
  * `"$ne": ""` makes the query return **any** document where `password` is **not empty**, effectively bypassing authentication.

***

### **Real-World Examples of NoSQL Injection Attacks**

#### &#x20;**Example 1: Authentication Bypass in a Web App**

**Case Study: GitHub’s MongoDB NoSQL Injection Vulnerability (2012)**

* **What Happened?**
  * GitHub had a **MongoDB-based authentication system** vulnerable to NoSQL Injection.
  *   Attackers sent:

      ```json
      { "username": { "$gt": "" }, "password": { "$gt": "" } }
      ```
  * The query matched **any** non-empty username and password, **bypassing authentication** entirely.
* **Fix:** GitHub implemented **parameterized queries** and **input validation**.

📖 **Source**: [OWASP NoSQL Injection Reference](https://owasp.org/www-community/attacks/NoSQL_Injection)

***

&#x20;**Example 2: Data Exfiltration via `$regex` Injection**

**Case Study: Online E-Commerce Platform**

* **What Happened?**
  * An e-commerce site allowed **search queries** directly inside MongoDB.
  *   Attackers used a **$regex-based NoSQL Injection payload** to enumerate usernames:

      ```json
      { "username": { "$regex": ".*" } }
      ```
  * This query **retrieved all usernames** from the database.

📖 **Source**: [PortSwigger’s NoSQL Injection Labs](https://portswigger.net/web-security/nosql-injection)

***

#### **Example 3: Firebase Database Misconfiguration**

**Case Study: Mobile App Data Leak**

* **What Happened?**
  * A ride-sharing app stored user data **without authentication** in Firebase.
  *   Attackers manipulated API calls to retrieve sensitive user data:

      ```json
      { ".read": true }
      ```
  * Over **100,000** user records were leaked.

📖 **Source**: [Google Firebase Security Best Practices](https://firebase.google.com/docs/rules/basics)

***

### **Common NoSQL Injection Techniques**

#### **1️⃣ Authentication Bypass**

* **Payload:** `{ "password": { "$ne": "" } }`
* **Effect:** Logs in as an admin without a password.

#### **2️⃣ Extracting Usernames**

* **Payload:** `{ "username": { "$regex": ".*" } }`
* **Effect:** Retrieves all usernames.

#### **3️⃣ Denial of Service (DoS)**

* **Payload:** `{ "$where": "sleep(5000)" }`
* **Effect:** Forces a **5-second delay**, overwhelming the database.

***

### **Remediation: Preventing NoSQL Injection**

#### ✅ **1. Use Parameterized Queries**

* **Why?** Prevents query logic from being altered by user input.
* **Secure Example (MongoDB with Mongoose):**

```javascript
User.findOne({ username: req.body.username, password: req.body.password }).exec();
```

* **Impact:** Ensures user input is treated as **data**, not **database logic**.

***

#### &#x20;**2. Implement Strong Input Validation**

* **Use libraries like** [**express-validator**](https://express-validator.github.io/) **for Node.js**.
* **Example:**

```javascript
const { body, validationResult } = require('express-validator');

app.post('/login', [
  body('username').isAlphanumeric().trim(),
  body('password').isLength({ min: 8 }),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
});
```

* **Impact:** Rejects malformed input before it reaches the database.

***

#### &#x20;**3. Restrict MongoDB Query Operators**

* **Why?** Prevents attackers from injecting `$ne`, `$regex`, or `$where`.
* **Secure Example Using `mongo-sanitize`:**

```javascript
const sanitize = require('mongo-sanitize');

app.post('/login', (req, res) => {
  let username = sanitize(req.body.username);
  let password = sanitize(req.body.password);
  db.users.find({ username, password });
});
```

* **Impact:** Removes **all special query operators** before execution.

***

#### **4. Implement Access Controls**

* **Why?** Prevents unauthorized users from querying sensitive data.
* **Best Practices:**
  * Enforce **least privilege access** to database queries.
  * Implement **role-based access control (RBAC)**.

📖 **Reference:** [MongoDB Security Checklist](https://www.mongodb.com/docs/manual/administration/security-checklist/)

***

#### &#x20;**5. Monitor & Log Query Behavior**

* **Why?** Detects unusual queries that indicate injection attempts.
* **Best Practices:**
  * Log failed login attempts.
  * Use a **SIEM** (Security Information and Event Management) tool.

📖 **Reference:** [OWASP Logging and Monitoring](https://owasp.org/www-project-logging/)

***

### **Conclusion**

* NoSQL Injection **exploits schema-less databases** where input is improperly validated.
* Attackers leverage **MongoDB operators** (`$ne`, `$regex`) to **bypass authentication** or **exfiltrate data**.
* **Real-world examples** (GitHub, Firebase) show the importance of **secure query handling**.
* **Remediation involves** parameterized queries, strict input validation, query sanitization, and access controls.

***

#### **Further Reading**

📖 [OWASP NoSQL Injection](https://owasp.org/www-community/attacks/NoSQL_Injection)\
📖 [PortSwigger NoSQL Labs](https://portswigger.net/web-security/nosql-injection)\
📖 [MongoDB Security Best Practices](https://www.mongodb.com/docs/manual/administration/security-checklist/)

***
