# XXE Attack Vectors Overview

## XXE Attack Vectors: Directory Listing, Malicious File Uploads, and Content-Type Manipulation

### **Overview**

**XML External Entity (XXE) vulnerabilities** can be exploited using various payloads depending on the target and desired outcome. **Directory listing payloads** help locate sensitive files, while **malicious file uploads** like SVGs can exploit improperly handled XML parsers. Applications often misrepresent accepted file types, creating an overlooked attack surface for potential XXE attacks.

***

### **Definition Bank**

| **Term**                           | **Definition**                                                                                    |
| ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| **XXE (XML External Entity)**      | A vulnerability where XML parsers mishandle external entities, allowing unauthorized file access. |
| **Directory Listing**              | A technique to list files in a directory, often used to identify sensitive information.           |
| **SVG (Scalable Vector Graphics)** | An XML-based image format that can be manipulated for XXE attacks.                                |
| **Content-Type**                   | Specifies the media type of the resource being sent, such as `image/png` or `application/xml`.    |
| **Client-Side Controls**           | Security checks performed on the client side that can be bypassed using tools like Burp Suite.    |

***

### **Key Concepts and Explanations**

#### **1. Directory Listing for Sensitive Files**

* **Directory listing** can help identify files containing hardcoded credentials, sensitive configurations, and SSH keys.
* Attackers exploit this to locate files for further XXE exploitation.

**Example Commands:**

```bash
# Listing the root directory
ls /

# Listing all files in the /etc directory
ls /etc
```

**Explanation:** The commands above provide visibility into the directory structure, potentially revealing sensitive files that attackers can exploit.

***

#### **2. SVG File Uploads for XXE Attacks**

* Some web applications allow the upload of SVG (**Scalable Vector Graphics**) files, which are XML-based.
* If XML parsing is not properly restricted, an attacker can include external entities within the SVG to exfiltrate data.

**Example of a Malicious SVG Payload:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY exfil SYSTEM "file:///etc/passwd"> ]>
<svg>
    <text>&exfil;</text>
</svg>
```

**Explanation:** This payload attempts to read the `/etc/passwd` file on a Unix system by embedding an external entity in the SVG file.

***

#### **3. Content-Type Manipulation and Client-Side Controls**

* **Content-Type Spoofing:** Some applications validate file types using the `Content-Type` header, which attackers can manipulate.
* **Bypassing Client-Side Controls:** Tools like Burp Suite allow attackers to modify upload requests, ignoring frontend restrictions.

**Example of a Manipulated Upload Request:**

```http
POST /upload HTTP/1.1
Host: example.com
Content-Type: image/svg+xml

<malicious_svg_payload>
```

**Explanation:** This payload bypasses frontend controls and submits a malicious SVG with a manipulated `Content-Type` header, allowing an attacker to execute an XXE exploit.

***

### **Comparisons and Alternatives**

| **Approach**                | **Secure?** | **Explanation**                                         |
| --------------------------- | ----------- | ------------------------------------------------------- |
| **Directory Listing**       | ❌           | Can reveal sensitive files if not properly restricted.  |
| **SVG Uploads**             | ❌           | Can lead to XXE attacks if not validated and sanitized. |
| **Content-Type Validation** | ✅           | Ensures uploaded files match expected formats.          |
| **JSON Parsing**            | ✅           | Does not support XML entities, reducing attack surface. |

***

### **Best Practices**

* **Disable External Entity Parsing:** Prevent XML parsers from resolving external entities.
* **Restrict File Upload Types:** Enforce strict server-side validation for accepted file formats.
* **Use Secure Parsers:** Opt for parsers that disable dangerous features by default.
* **Monitor Content Types:** Verify the `Content-Type` server-side to prevent manipulation.
* **Regular Security Audits:** Perform code reviews and dynamic testing to identify XXE vulnerabilities.

***

### **Example of Secure XML Parsing (Python)**

```python
import defusedxml.ElementTree as ET
ET.parse('data.xml')  # Secure parsing without external entities
```

**Explanation:** The `defusedxml` library prevents external entity parsing, mitigating the risk of XXE attacks effectively.

***

### **Conclusion**

Understanding **directory listing techniques, SVG-based XXE payloads, and content-type manipulation** is essential to securing XML parsers and preventing unauthorized file access. By enforcing **secure XML processing**, restricting **file uploads**, and validating **content types**, organizations can significantly reduce the risk of XXE exploitation.
