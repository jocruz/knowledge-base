# SQL Injection (SQLi)

## **SQL Injection (SQLi) – Comprehensive Overview**

***

### **Introduction to SQL Injection**

SQL Injection (SQLi) is a **critical security vulnerability** that allows attackers to manipulate SQL queries executed by an application’s database. By injecting **malicious SQL code** into input fields, attackers can:

* **Bypass authentication mechanisms**
* **Extract, modify, or delete sensitive data**
* **Compromise the integrity of the database**
* **Perform denial-of-service (DoS) attacks**
* **Potentially escalate privileges and execute system commands**

SQLi has been consistently ranked among the **OWASP Top 10 security risks**, making it a **high-priority concern** for web application security.

***

### **How SQL Injection Works**

SQL Injection exploits occur when **user-controlled input** is directly **concatenated into an SQL query** without **proper validation or sanitization**. This **breaks the principle** of separating data and executable code, allowing an attacker to execute arbitrary SQL commands.

#### **Example of Vulnerable Code**

```php
$query = "SELECT * FROM users WHERE username = '" + $user_input + "'";
```

**If the attacker inputs:**

```sql
' OR 1=1--  
```

**The resulting query becomes:**

```sql
SELECT * FROM users WHERE username = '' OR 1=1--
```

Since `1=1` always evaluates to **TRUE**, the query **returns all users**, effectively **bypassing authentication**.

***

### **Types of SQL Injection Attacks**

#### **1. Classic SQL Injection (Authentication Bypass)**

Occurs when unvalidated input **directly alters** a database query, allowing attackers to manipulate data retrieval.

**Example:**

```sql
SELECT * FROM users WHERE username = '' OR 1=1-- AND password = '';
```

***

#### **2. Union-Based SQL Injection**

Uses the **UNION operator** to retrieve data from **additional tables** within the database.

**Example:** Extracting **usernames and passwords**

```sql
' UNION SELECT username, password FROM users--
```

***

#### **3. Error-Based SQL Injection**

Relies on **database error messages** to extract information about the database structure.

**Example:**

```sql
' ORDER BY 100--
```

If an error occurs, it indicates that the **database has fewer than 100 columns**, helping the attacker determine the **schema structure**.

***

#### **4. Blind SQL Injection**

Occurs when **no direct feedback** is provided, but the **database behavior changes** based on injected queries.

**Example: Boolean-Based SQLi**

```sql
' AND 1=1--  (Valid request)  
' AND 1=0--  (Invalid request)  
```

By observing **application behavior**, an attacker can infer if the query executed successfully.

***

#### **5. Time-Based SQL Injection**

Uses SQL functions like `SLEEP()` to **delay responses** and infer success.

**Example:**

```sql
' OR IF(1=1, SLEEP(5), 0)--
```

If the response is **delayed by 5 seconds**, the database executed the command, confirming the vulnerability.

***

### **Real-World SQL Injection Incidents**

#### **1. Sony PlayStation Network Breach (2011)**

* **Attack:** SQL Injection exploited to **compromise 77 million user accounts**.
* **Impact:** Attackers stole **personal and financial data**, including **credit card details**.
* **Lesson Learned:** Sony **lacked parameterized queries** and **input validation**, leading to massive data exposure.
* **Reference:** [Sony PSN SQL Injection Breach](https://www.csoonline.com/article/2130877/sony-data-breach-history.html)

***

#### **2. TalkTalk Data Breach (2015)**

* **Attack:** Hackers used **blind SQL injection** to exfiltrate **4 million customer records**.
* **Impact:** **Financial and personal information leaked**, leading to **£400,000 in fines**.
* **Lesson Learned:** Poor **input validation** and **database security controls** enabled the attack.
* **Reference:** [TalkTalk SQLi Attack](https://www.bbc.com/news/technology-34589275)

***

#### **3. Heartland Payment Systems Attack (2008)**

* **Attack:** SQL Injection was used to install **malware** on financial transaction systems.
* **Impact:** Over **130 million credit card records** were stolen.
* **Lesson Learned:** The company had **insufficient security monitoring** and **access controls**.
* **Reference:** [Heartland Payment Breach](https://www.computerworld.com/article/2525786/heartland-payment-systems--breach-determined-to-be-sql-injection-attack.html)

***

### **Preventing SQL Injection**

#### **1. Use Parameterized Queries**

Using **parameterized queries** ensures user input is treated as **data**, not **executable SQL code**.

**Secure Example (PHP & MySQLi):**

```php
$stmt = $connection->prepare("SELECT * FROM users WHERE username = ?");
$stmt->bind_param("s", $user_input);
$stmt->execute();
```

***

#### **2. Use Prepared Statements**

Prepared statements **separate SQL code from user input**, preventing SQL injection attacks.

**Secure Example (Python & MySQL):**

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (user_input,))
```

***

#### **3. Validate User Input**

Restrict inputs using **whitelists**, **regex validation**, and **strict data typing**.

**Secure Example (Python - Input Validation with Regex):**

```python
import re

if not re.match(r'^[a-zA-Z0-9_]+$', username):
    raise ValueError("Invalid input")
```

***

#### **4. Implement the Principle of Least Privilege (PoLP)**

Ensure database accounts have **minimal permissions** to reduce damage in case of an SQLi attack.

**Example: Restricting User Permissions (MySQL):**

```sql
GRANT SELECT ON database.users TO 'app_user'@'localhost';
```

This prevents the application user from **modifying or deleting sensitive data**.

***

#### **5. Disable Error Messages in Production**

Avoid leaking database structure details through error messages.

**Example: Restricting SQL Errors (MySQL):**

```sql
SET SESSION sql_mode = 'NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

***

### **Industry Standards & References**

| **Reference**                                       | **Link**                                                                    |
| --------------------------------------------------- | --------------------------------------------------------------------------- |
| **OWASP SQL Injection Guide**                       | [OWASP SQLi Guide](https://owasp.org/www-community/attacks/SQL_Injection)   |
| **OWASP Top 10 (2021) - Injection Vulnerabilities** | [OWASP Top 10](https://owasp.org/www-project-top-ten/)                      |
| **PortSwigger SQL Injection Labs**                  | [PortSwigger SQLi Labs](https://portswigger.net/web-security/sql-injection) |
| **NIST Secure Software Development Framework**      | [NIST Guide](https://csrc.nist.gov/publications/detail/sp/800-218/final)    |

***

### **Final Thoughts**

**SQL Injection remains one of the most severe web vulnerabilities** due to its **widespread impact**. Attackers can exploit **improperly handled user input** to compromise **databases, extract sensitive information, or even take full control of a system**.

To **mitigate** SQLi risks:

**Use parameterized queries and prepared statements**\
I**mplement input validation and sanitization**\
**Enforce least privilege access control**\
**Disable error messages in production**\
**Regularly conduct penetration testing and security audits**

By implementing **secure coding practices**, organizations can **effectively mitigate SQL Injection attacks** and ensure **robust database security**.
