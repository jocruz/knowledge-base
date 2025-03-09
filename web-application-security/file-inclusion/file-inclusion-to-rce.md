# File Inclusion to RCE

## Achieving Remote Code Execution (RCE) with File Inclusion Vulnerabilities

### **Overview**

**File Inclusion Vulnerabilities** can sometimes be escalated to achieve **Remote Code Execution (RCE)** depending on the server’s configuration and available attack surfaces. Attackers leverage file inclusion flaws to execute arbitrary commands on the target system. The most common techniques include:

* **Log File Poisoning**: Injecting a malicious payload into server logs and executing it via file inclusion.
* **Uploading a Web Shell**: Uploading a script (e.g., `.php` file) and executing it through file inclusion.
* **Proc Self Environment Technique**: Injecting code into `/proc/self/environ` and executing it via file inclusion.
* **SSH Key Theft**: Extracting SSH keys from compromised file paths for unauthorized access.

***

### **Definition Bank**

| **Term**                  | **Definition**                                                  | **Example**                                              |
| ------------------------- | --------------------------------------------------------------- | -------------------------------------------------------- |
| **Log File Poisoning**    | Injecting a payload into server logs and executing via LFI.     | Placing a PHP payload in the `User-Agent` header.        |
| **Web Shell Upload**      | Uploading a malicious script and executing via file inclusion.  | Uploading `shell.php` and including it with LFI.         |
| **Proc Self Environment** | Injecting code into `/proc/self/environ` and executing via LFI. | Including `/proc/self/environ` with an injected payload. |
| **SSH Key Theft**         | Extracting SSH keys for unauthorized access.                    | Stealing `.ssh/id_rsa` for remote login.                 |

***

### **Key Techniques for RCE via File Inclusion**

#### **Log File Poisoning**

* **Method:**
  * Inject a payload into HTTP headers (e.g., `User-Agent`).
  * The payload is written to the server logs.
  * Exploit file inclusion to reference the log file and execute the payload.

**Example Payload:**

```plaintext
GET /index.php?file=/var/log/apache/access.log
User-Agent: <?php system($_GET['cmd']); ?>
```

* **Consideration:** This technique is more common in **Capture The Flag (CTF)** environments but can occur in misconfigured production systems.

***

#### **Web Shell Upload**

* **Method:**
  * Upload a **PHP web shell** using an unprotected file upload feature.
  * Use a **file inclusion vulnerability** to execute the uploaded shell.

**Example Payload:**

```plaintext
http://localhost/index.php?file=uploads/shell.php
```

* **Impact:**
  * Allows **arbitrary command execution** on the server.
  * Attackers can escalate privileges if writable directories are present.

***

#### **Proc Self Environment Technique**

* **Method:**
  * Inject a **malicious payload** into `/proc/self/environ` using an HTTP header.
  * Include `/proc/self/environ` via LFI to trigger execution.

**Example Payload:**

```plaintext
GET /index.php?file=/proc/self/environ
User-Agent: <?php system($_GET['cmd']); ?>
```

* **Consideration:**
  * Some modern Linux distributions restrict access to `/proc/self/environ`.

***

#### **SSH Key Theft**

* **Method:**
  * Exploit LFI to **retrieve private SSH keys** from compromised servers.

**Example Payload:**

```plaintext
GET /index.php?file=/home/user/.ssh/id_rsa
```

* **Impact:**
  * An attacker can use the stolen private key to **gain SSH access**.

***

### **Testing and Mitigation Strategies**

#### **Testing Checklist**

✔ Check if the application reflects **user-controlled input** in file paths. ✔ Attempt to include sensitive files (e.g., `/etc/passwd`). ✔ Use encoding techniques (**Base64**, **URL encoding**) to bypass filters. ✔ Test log poisoning by injecting **payloads into headers**. ✔ Try various **PHP wrappers** (e.g., `php://input`, `php://filter`).

***

#### **Mitigation Techniques**

🔒 **Validate and Sanitize User Inputs**: Restrict file inclusion to a predefined list of allowed files. 🔒 **Use an Allowlist Approach**: Block unauthorized file paths explicitly. 🔒 **Disable Dangerous PHP Functions**: Disable `include()`, `require()`, and `allow_url_include` in production environments. 🔒 **Restrict Directory Access**: Use `open_basedir` or a chroot jail to limit access to system files. 🔒 **Harden Server Permissions**: Ensure web servers run with **least privilege access**. 🔒 **Log and Monitor Suspicious Activity**: Set up alerts for **unexpected file inclusion attempts**.

***

### **Key Takeaways**

✔ **Remote Code Execution (RCE) through file inclusion** often relies on **secondary misconfigurations**. ✔ **Even without RCE, file inclusion vulnerabilities** can expose **sensitive data**. ✔ **Systematic testing and proper input validation** are critical to mitigating these risks.

***
