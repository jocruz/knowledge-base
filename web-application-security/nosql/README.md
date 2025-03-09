# NoSql

## NoSQL Injection – Comprehensive Overview

***

### Introduction to NoSQL Injection

NoSQL Injection is a security vulnerability that occurs when unvalidated user input is inserted into NoSQL database queries, allowing attackers to manipulate query logic. Unlike SQL Injection, which targets relational databases (MySQL, PostgreSQL, MSSQL), NoSQL Injection exploits schema-flexible databases such as MongoDB, CouchDB, Firebase, and Elasticsearch.

#### Potential Risks of NoSQL Injection

* Authentication Bypass – Attackers can log in without valid credentials.
* Data Exfiltration – Attackers can extract confidential user records.
* Data Manipulation – Attackers can modify or delete sensitive documents.
* Denial of Service (DoS) – Large queries can overload and crash the database.

***

### How NoSQL Injection Works

Unlike traditional SQL, NoSQL databases process queries as JSON-like objects. A failure to validate or sanitize input allows attackers to inject special query operators (`$ne`, `$gt`, `$regex`) into API endpoints or login forms.

#### Example: Vulnerable Code (MongoDB Query in JavaScript/Node.js)

```javascript
db.users.find({ username: req.body.username, password: req.body.password });
```

#### Why is this Vulnerable?

* The application directly accepts user input from `req.body.username` and `req.body.password` without sanitization.
* An attacker can modify this input to alter the query logic.

#### Payload for Authentication Bypass

```json
{
  "username": "admin",
  "password": { "$ne": "" }
}
```

**Impact:**

* The `$ne` (not equal) operator forces the database to return any document where the password is not empty, effectively bypassing authentication.

***

### Real-World Examples of NoSQL Injection Attacks

#### 1. Authentication Bypass in GitHub (2012)

* Case Study: GitHub’s MongoDB NoSQL Injection Vulnerability
* What Happened?
  * GitHub had a MongoDB-based authentication system that was vulnerable to NoSQL Injection.
  *   Attackers sent the following query:

      ```json
      { "username": { "$gt": "" }, "password": { "$gt": "" } }
      ```
  * The query matched any non-empty username and password, bypassing authentication entirely.
* Fix: GitHub implemented parameterized queries and input validation.
* Source: [OWASP NoSQL Injection Reference](https://owasp.org/www-community/attacks/NoSQL_Injection)

***

#### 2. Data Exfiltration via `$regex` Injection

* Case Study: E-Commerce Platform Attack
* What Happened?
  * An e-commerce website allowed search queries directly inside MongoDB.
  *   Attackers used a `$regex`-based NoSQL Injection payload to enumerate usernames:

      ```json
      { "username": { "$regex": ".*" } }
      ```
  * This query retrieved all usernames from the database.
* Source: [PortSwigger NoSQL Injection Labs](https://portswigger.net/web-security/nosql-injection)

***

#### 3. Firebase Database Misconfiguration

* Case Study: Mobile App Data Leak
* What Happened?
  * A ride-sharing app stored user data without authentication in Firebase.
  *   Attackers manipulated API calls to retrieve sensitive user data:

      ```json
      { ".read": true }
      ```
  * Over 100,000 user records were leaked due to a misconfigured Firebase database.
* Source: [Google Firebase Security Best Practices](https://firebase.google.com/docs/rules)

***

### Common NoSQL Injection Techniques

| Technique               | Payload Example                      | Effect                                              |
| ----------------------- | ------------------------------------ | --------------------------------------------------- |
| Authentication Bypass   | `{ "password": { "$ne": "" } }`      | Logs in as an admin without a password.             |
| Extracting Usernames    | `{ "username": { "$regex": ".*" } }` | Retrieves all usernames.                            |
| Denial of Service (DoS) | `{ "$where": "sleep(5000)" }`        | Forces a 5-second delay, overwhelming the database. |

***

### Remediation: Preventing NoSQL Injection

#### 1. Use Parameterized Queries

Why? Prevents query logic from being altered by user input.

**Secure Example (MongoDB with Mongoose in Node.js)**

```javascript
User.findOne({ username: req.body.username, password: req.body.password }).exec();
```

Impact:

* Ensures user input is treated as data, not executable database logic.

***

#### 2. Implement Strong Input Validation

* Use validation libraries like `express-validator` for Node.js applications.
* Ensure inputs match expected data types (e.g., restrict usernames to alphanumeric characters).

**Secure Example (Node.js using express-validator)**

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

Impact:

* Rejects malformed input before it reaches the database.

***

#### 3. Restrict MongoDB Query Operators

Why? Prevents attackers from injecting `$ne`, `$regex`, or `$where`.

**Secure Example Using `mongo-sanitize` in Node.js**

```javascript
const sanitize = require('mongo-sanitize');

app.post('/login', (req, res) => {
  let username = sanitize(req.body.username);
  let password = sanitize(req.body.password);
  db.users.find({ username, password });
});
```

Impact:

* Removes special query operators before execution.

***

#### 4. Implement Access Controls

Why? Prevents unauthorized users from querying sensitive data.

**Best Practices:**

* Enforce least privilege access to database queries.
* Implement role-based access control (RBAC) to restrict database interactions.

Source: [MongoDB Security Checklist](https://www.mongodb.com/docs/manual/security-checklist/)

***

#### 5. Monitor & Log Query Behavior

Why? Detects unusual queries that indicate injection attempts.

**Best Practices:**

* Log failed login attempts to detect brute force attacks.
* Use a SIEM (Security Information and Event Management) tool to monitor queries.

Source: [OWASP Logging and Monitoring](https://owasp.org/www-community/controls/Logging_and_monitoring)

***

### Final Thoughts

* NoSQL Injection exploits schema-less databases where input is improperly validated.
* Attackers leverage MongoDB operators (`$ne`, `$regex`) to bypass authentication or exfiltrate data.
* Real-world examples (GitHub, Firebase) highlight the importance of secure query handling.
* Remediation involves:
  * Parameterized queries
  * Strict input validation
  * Query sanitization
  * Access controls

By following secure coding best practices, regularly testing for vulnerabilities, and monitoring query behavior, organizations can mitigate the risks associated with NoSQL Injection attacks.

***

### Further Reading

[OWASP NoSQL Injection Guide](https://owasp.org/www-community/attacks/NoSQL_Injection)\
[PortSwigger NoSQL Injection Labs](https://portswigger.net/web-security/nosql-injection)\
[MongoDB Security Best Practices](https://www.mongodb.com/docs/manual/security/)
