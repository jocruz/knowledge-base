# File Path Traversal (Portswigger Lab)

## File Inclusion Vulnerabilities - Lab Analysis

### **Lab: File Path Traversal - Simple Case**

#### **Lab URL**:

[https://portswigger.net/web-security/file-path-traversal/lab-simple](https://portswigger.net/web-security/file-path-traversal/lab-simple)

#### **Objective**

The goal of this lab is to exploit a **Path Traversal vulnerability** to retrieve the contents of the `/etc/passwd` file. This exercise demonstrates how insecure handling of file paths in a web application can be leveraged to gain unauthorized access to sensitive system files.

#### **Understanding the Vulnerability**

* The web application dynamically includes files based on user input but fails to properly validate or sanitize the `filename` parameter.
* This oversight allows an attacker to manipulate the file path, using directory traversal sequences (`../`) to access restricted files.
* In this lab, we will use **Burp Suite** to analyze and manipulate HTTP requests, leveraging path traversal techniques to read sensitive files from the server.

***

### **Step-by-Step Solution**

#### **Step 1: Generate Traffic**

* Navigate to the lab and interact with the web application to observe how image files are retrieved.
*   Identify a GET request similar to the following:

    ```plaintext
    GET /images?filename=67.png HTTP/1.1
    Host: target-website.com
    ```
* The `filename` parameter dynamically specifies which image file is loaded.

***

#### **Step 2: Intercept and Modify the Request**

* Open **Burp Suite** and enable the **Intercept** feature.
* Capture the request containing the `filename` parameter.
*   Modify the `filename` parameter to attempt directory traversal:

    ```plaintext
    GET /images?filename=../../../etc/passwd HTTP/1.1
    Host: target-website.com
    ```
* Forward the modified request to the server and observe the response.

***

#### **Step 3: Verify Successful Exploitation**

*   If successful, the server will return the contents of `/etc/passwd`, which contains system user information:

    ```plaintext
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    ```
* This confirms that the application is vulnerable to **Path Traversal**, as it allows unauthorized access to system files.

***

### **Alternative Approach: Automating with Wordlists**

For more advanced testing, security professionals often use predefined wordlists to identify vulnerable file inclusion points. In a real-world scenario:

1.  **Locate LFI Wordlists**:

    ```bash
    find /usr/share/seclists -iname "*lfi*"
    ```
2. **Use Burp Suite Intruder to Test Multiple Payloads**:
   *   Load a payload list such as:

       ```plaintext
       /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
       ```
   * Automate the injection of directory traversal payloads to identify exploitable endpoints.

***

### **Why This is an Example of Path Traversal**

#### **Characteristics of the Vulnerability**

1. **Lack of Input Validation**: The application allows user-controlled input in file paths without sanitization.
2. **Use of Dynamic File Inclusion**: The `filename` parameter is used directly in the file retrieval process, making it susceptible to manipulation.
3. **Exploitation via Directory Traversal Sequences**: The attacker can navigate the server’s directory structure using `../` to access restricted files.

#### **Security Implications**

* **Information Disclosure**: Exposing system files like `/etc/passwd` can help attackers enumerate valid usernames, which may be leveraged for further attacks (e.g., brute-force login attempts).
* **Potential Remote Code Execution**: If an application allows **file write access**, an attacker might escalate from reading files to executing arbitrary scripts.

***

### **Remediation Strategies**

1. **Sanitize User Input**
   * Implement strict validation for file paths, restricting input to expected values.
   * Use an allowlist approach to limit acceptable filenames.
2. **Restrict File Access**
   * Ensure the application can only access files within designated directories.
   * Use security mechanisms like `chroot` or `open_basedir` to sandbox file operations.
3. **Monitor and Log Suspicious Activity**
   * Implement logging for unauthorized access attempts.
   * Set up alerts for unusual file access patterns to detect exploitation attempts.
4. **Use Secure File Inclusion Methods**
   * Instead of dynamically including files based on user input, hardcode safe file paths where possible.
   * Disable directory traversal sequences by properly resolving paths before file inclusion.

***

### **Conclusion**

This lab demonstrates how insecure file inclusion mechanisms lead to **Path Traversal vulnerabilities**, enabling unauthorized access to sensitive system files. Proper input validation, directory access restrictions, and security monitoring are critical to mitigating these risks. Organizations should regularly perform security assessments to detect and remediate such vulnerabilities before attackers exploit them.
