# Blind SQL Injection Using Time Delays (PortSwigger Lab)

**Blind SQL Injection Using Time Delays**\
[Lab Link](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)
------------------------------------------------------------------------------------

***

### **High-Level Summary**

This lab focuses on **blind SQL injection using time delays** to infer database behavior. Since no direct output is returned, attackers rely on **response time differences** to determine whether an injection is successful. The objective is to manipulate a SQL query via the `TrackingId` cookie to introduce a **10-second delay**, proving the vulnerability.

***

### **Key Concepts & Definitions**

| Term                                 | Definition                                                                                                         |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| **Blind SQL Injection (Time-Based)** | A SQLi technique where an attacker infers database behavior based on response time rather than direct output.      |
| **Time Delay Functions**             | Database-specific functions that pause query execution (e.g., `pg_sleep(10)` in PostgreSQL, `SLEEP(10)` in MySQL). |
| **Synchronous Query Execution**      | The database processes a request before returning a response, allowing measurable delays.                          |
| **pg\_sleep(10)**                    | PostgreSQL function that causes the database to pause execution for 10 seconds.                                    |
| **SLEEP(10)**                        | MySQL function that introduces a delay of 10 seconds.                                                              |
| **WAITFOR DELAY**                    | Microsoft SQL Server command that delays execution (e.g., `WAITFOR DELAY '0:0:10'`).                               |
| **dbms\_pipe.receive\_message**      | Oracle function used to delay query execution (e.g., `dbms_pipe.receive_message(('a'),10)`).                       |

***

### **Lab Overview (PortSwigger)**

The application uses a `TrackingId` cookie value in a SQL query without sanitization. Since results are not directly visible, we exploit the SQL query’s **synchronous execution** by injecting a **time delay** and measuring the server response time to confirm SQL injection.

#### **Lab Goal:**

Introduce a **10-second delay** using SQLi payloads to verify the vulnerability.

***

### **PortSwigger Official Solution**

1. **Intercept the `TrackingId` cookie** using Burp Suite.
2.  **Inject a time delay** using the PostgreSQL function:

    ```sql
    TrackingId=x'||pg_sleep(10)--
    ```
3. **Send the request** and observe a 10-second delay, confirming SQL injection.
4. **Lab is solved** once the delay is successfully introduced.

***

### **Instructor Notes: Comprehensive Walkthrough**

#### **Step 1: Identifying SQL Injection Possibilities**

* The lab’s description mentions **blind SQLi using time delays**, so we focus on **time-based payloads**.
* Without this hint, we would have tried various **Boolean-based SQLi** techniques first.

#### **Step 2: Selecting the Right Time Delay Function**

Consulting the **PortSwigger SQL Injection Cheat Sheet**, we identify:

| Database   | Delay Payload Example                 |
| ---------- | ------------------------------------- |
| Oracle     | `dbms_pipe.receive_message(('a'),10)` |
| Microsoft  | `WAITFOR DELAY '0:0:10'`              |
| PostgreSQL | `SELECT pg_sleep(10)`                 |
| MySQL      | `SELECT SLEEP(10)`                    |

For this lab, **PostgreSQL functions** (`pg_sleep(10)`) are applicable.

#### **Step 3: Constructing the Payload**

We modify the `TrackingId` cookie:

```sql
TrackingId=x'||pg_sleep(10)--
```

* The `||` operator is used for **string concatenation in PostgreSQL**.
* `pg_sleep(10)` forces the database to pause execution for **10 seconds**.
* The `--` comments out the rest of the query, preventing syntax errors.

#### **Step 4: Sending the Request and Measuring Delay**

* Sending the modified request results in a **response time of approximately 20 seconds**, not 10.
* This suggests the query executes twice, likely in multiple parts of the application.

#### **Step 5: Confirming Consistency**

* Resending the request still results in a **consistent delay**, confirming SQL injection.
* Even though **20 seconds** was unexpected, it still proves the vulnerability.

***

### **Methodology for Detecting Time-Based Blind SQLi**

#### **1. Identify Blind SQLi Indicators**

* No direct output from queries.
* Suspicious changes in response time.
* Unusual processing behavior.

#### **2. Select Database-Specific Delay Functions**

* **PostgreSQL:** `pg_sleep(10)`
* **MySQL:** `SLEEP(10)`
* **SQL Server:** `WAITFOR DELAY '0:0:10'`
* **Oracle:** `dbms_pipe.receive_message(('a'),10)`

#### **3. Inject Time-Based Payloads**

*   **Basic Test:**

    ```sql
    ' || pg_sleep(10) --
    ```
*   **Alternative (MySQL):**

    ```sql
    ' OR SLEEP(10) --
    ```

#### **4. Observe Response Delays**

* If the **server takes 10+ seconds longer** to respond, SQLi is confirmed.

#### **5. Iterate and Extract Data**

* Using **binary search techniques**, extract database names, table names, and user credentials.
*   Example for guessing database name:

    ```sql
    ' || IF((SELECT SUBSTRING(database(),1,1)='a'), SLEEP(5), 0) --
    ```

***

### **Troubleshooting Tips**

* If `pg_sleep(10)` fails, try **alternative database functions**.
* Ensure the **correct comment syntax** (`--`, `#`, `/* */`) is used for the DB type.
* Use Burp’s **Repeater** tool to **compare response times**.
* If delays stack, check if the query executes multiple times per request.

***

### **Key Takeaways**

* **Time-based SQLi** allows attackers to confirm vulnerabilities **without direct output**.
* Knowing **database-specific time functions** speeds up exploitation.
* **Delays may stack** if multiple queries execute with the injected payload.
* **Use time delays to infer data** via binary search techniques.

***

### **Next Steps for Exam & Real-World Readiness**

* **Practice:** Repeat this lab with different databases (MySQL, MSSQL, Oracle).
* **Automate:** Use `sqlmap` to confirm time-based SQLi.
* **Cheat Sheet:** Keep a reference list of **database-specific sleep functions**.
* **Further Learning:** Study **blind SQLi using Boolean conditions** and **error-based SQLi**.

***

