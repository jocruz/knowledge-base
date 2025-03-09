# SQL Injection (SQLi) Overview

## SQL Injection (SQLi) Overview

#### **Introduction to SQL Injection**

SQL Injection (SQLi) is a **critical security vulnerability** that allows attackers to manipulate SQL queries executed by an application’s database. By injecting malicious SQL code into input fields, attackers can:

* **Bypass authentication mechanisms**.
* **Extract, modify, or delete sensitive data**.
* **Compromise the integrity of the database**.
* **Perform denial-of-service (DoS) attacks**.
* **Potentially escalate privileges and execute system commands**.

#### **How SQL Injection Works**

SQLi exploits occur when **user-controlled input is directly concatenated into an SQL query** without proper validation or sanitization. This breaks the principle of separating data and code, allowing an attacker to execute arbitrary SQL commands.

**Example of Vulnerable Code**

```sql
$query = "SELECT * FROM users WHERE username = '" + $user_input + "'";
```

* **If `$user_input` contains:** `' OR 1=1--`,
* **The resulting query becomes:**

```sql
SELECT * FROM users WHERE username = '' OR 1=1--
```

Since `1=1` always evaluates to `TRUE`, **all users are returned, effectively bypassing authentication**.

***

### **Types of SQL Injection**

#### **1. Classic SQL Injection**

Occurs when unvalidated input directly alters a database query, allowing attackers to manipulate data retrieval.

**Example:** Authentication bypass

```sql
SELECT * FROM users WHERE username = '' OR 1=1-- AND password = '';
```

#### **2. Union-Based SQL Injection**

Uses the `UNION` operator to retrieve data from additional tables within the database.

**Example:** Extracting usernames and passwords

```sql
' UNION SELECT username, password FROM users--
```

#### **3. Error-Based SQL Injection**

Relies on database error messages to extract information about the database structure.

**Example:**

```sql
' ORDER BY 100--
```

If an error occurs, it indicates fewer than 100 columns, helping the attacker determine the schema.

#### **4. Blind SQL Injection**

Occurs when no direct feedback is provided but the database behavior changes based on injected queries.

**Example:** Boolean-based SQLi

```sql
' AND 1=1--  (Valid request)
' AND 1=0--  (Invalid request)
```

#### **5. Time-Based SQL Injection**

Uses SQL functions like `SLEEP()` to delay responses and infer success.

**Example:**

```sql
' OR IF(1=1, SLEEP(5), 0)--
```

If the response is delayed by 5 seconds, the database executed the command, confirming the vulnerability.

***

### **Preventing SQL Injection**

#### **1. Use Parameterized Queries**

Using parameterized queries ensures user input is treated as data rather than executable SQL code.

**Secure Example (PHP & MySQLi):**

```php
$stmt = $connection->prepare("SELECT * FROM users WHERE username = ?");
$stmt->bind_param("s", $user_input);
$stmt->execute();
```

#### **2. Use Prepared Statements**

Prepared statements ensure queries are compiled separately from input values.

**Secure Example (Python & MySQL):**

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (user_input,))
```

#### **3. Validate User Input**

Restrict inputs using whitelists, regex validation, and strict data typing.

```python
import re
if not re.match(r'^[a-zA-Z0-9_]+$', username):
    raise ValueError("Invalid input")
```

#### **4. Implement Least Privilege Principle**

Ensure database accounts have minimal permissions:

```sql
GRANT SELECT ON database.users TO 'app_user'@'localhost';
```

#### **5. Disable Error Messages in Production**

Configure databases to avoid leaking information through errors:

```sql
SET SESSION sql_mode = 'NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

***

### **Real-World SQL Injection Incidents**

#### **1. Sony PlayStation Network (2011)**

* Attackers exploited SQLi to compromise **77 million accounts**.
* **Impact:** Personal data, including credit card details, was stolen.
* **Lesson Learned:** Sony lacked **parameterized queries and input validation**.

#### **2. TalkTalk Data Breach (2015)**

* Hackers used **blind SQL injection** to exfiltrate **4 million customer records**.
* **Impact:** Financial and personal information leaked.
* **Lesson Learned:** Lack of **input validation** and **database security controls**.

#### **3. Heartland Payment Systems (2008)**

* Attackers used **SQLi to install malware** on financial transaction systems.
* **Impact:** Over **130 million credit card records** were stolen.
* **Lesson Learned:** Insufficient **security monitoring** and **access controls**.

***

### **Industry Standards & References**

#### **1. OWASP SQL Injection Guide**

[https://owasp.org/www-community/attacks/SQL\_Injection](https://owasp.org/www-community/attacks/SQL_Injection)

#### **2. OWASP Top 10 (2021) - Injection Vulnerabilities**

[https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)

#### **3. PortSwigger SQL Injection Labs**

[https://portswigger.net/web-security/sql-injection](https://portswigger.net/web-security/sql-injection)

#### **4. NIST Secure Software Development Framework**

[https://csrc.nist.gov/publications/detail/sp/800-218/final](https://csrc.nist.gov/publications/detail/sp/800-218/final)

***

### **Final Thoughts**

SQL Injection remains one of the **most severe web vulnerabilities**, with widespread impact. **Implementing parameterized queries, input validation, and the principle of least privilege** are essential defenses. Security teams should also conduct **regular penetration testing** and **secure coding reviews** to mitigate risks effectively.
