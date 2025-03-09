# XXE

## **XML External Entity (XXE) Vulnerabilities – Comprehensive Overview**

***

### **Introduction to XXE Attacks**

Extensible Markup Language (**XML**) is widely used for **data exchange** between applications due to its **structured format** and **flexibility**. However, **poorly configured XML parsers** can introduce **XML External Entity (XXE) vulnerabilities**, which allow attackers to:

* **Access sensitive local files** on the server.
* **Execute server-side request forgery (SSRF)** attacks.
* **Leak environment variables and internal configurations.**
* **Perform denial-of-service (DoS) attacks** using recursive entity expansion (billion laughs attack).

#### **Why is XXE Dangerous?**

* **Risk:** XXE can lead to **data breaches, privilege escalation, and full server compromise**.
* **Impact:** Attackers can extract **sensitive files**, **query internal services**, or **launch DoS attacks**.
* **Mitigation:** Proper **secure XML parsing techniques** prevent attackers from exploiting XML entities.

***

### **Definition Bank**

| **Term**                               | **Definition**                                                                                                                         |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **XML External Entity (XXE)**          | A vulnerability that allows attackers to exploit the way XML parsers process external entities, leading to unauthorized access.        |
| **Server-Side Request Forgery (SSRF)** | A type of attack where a server is manipulated into making unintended requests to internal or external resources.                      |
| **Billion Laughs Attack**              | A recursive entity expansion attack that causes **denial-of-service (DoS)** by exhausting system memory.                               |
| **DTD (Document Type Definition)**     | A set of rules that define the structure of an XML document, which can be exploited if external entities are allowed.                  |
| **Blind XXE**                          | An XXE attack where the attacker cannot directly see the response but can still exfiltrate data through time delays or error messages. |

***

### **Key Concepts and Explanations**

#### **1. How XML Works**

XML follows a **tree-structured format**, allowing data to be represented hierarchically.

**Example of a Simple XML Document:**

```xml
<user>
    <username>admin</username>
    <email>admin@example.com</email>
</user>
```

***

#### **2. What is an External Entity?**

**Entities** in XML act as **placeholders** for data. **External entities** reference an **external file** or **resource**, making them vulnerable if **not properly controlled**.

**Example of a Safe Local Entity Definition:**

```xml
<!DOCTYPE foo [
    <!ENTITY example "Hello, World!">
]>
<root>&example;</root>
```

***

#### **3. How an XXE Attack Works**

Attackers **inject a malicious external entity** into an XML document, forcing the parser to access restricted files.

**Exploitable External Entity Example:**

```xml
<!DOCTYPE foo [
    <!ENTITY exfil SYSTEM "file:///etc/passwd">
]>
<root>&exfil;</root>
```

**Why This is Dangerous:**

* The external entity **exfil** references a **local file** (`/etc/passwd`).
* If the **XML parser processes this**, it **retrieves and exposes** the file's contents.

***

### **Real-World XXE Exploits**

#### **1. Uber Bug Bounty Disclosure (2019)**

* **Attack Vector:** A security researcher **exploited an XXE vulnerability** to retrieve **AWS metadata**, leading to **server-side request forgery (SSRF)** and **internal resource access**.
* **Impact:** Attackers could **query AWS metadata services**, potentially leading to **full server compromise**.
* **Lesson Learned:** Disabling **external entity resolution** prevents such attacks.
* **Source:** [HackerOne Bug Bounty Report](https://hackerone.com/reports/408693)

***

#### **2. Samsung SmartThings XXE Attack**

* **Attack Vector:** Attackers exploited **XXE in XML-based APIs**, leading to **sensitive data leaks**.
* **Impact:** Unauthorized access to **user credentials, internal APIs, and IoT device data**.
* **Lesson Learned:** Secure **XML processing and input validation** are critical for protecting IoT services.
* **Source:** [Black Hat Conference Report](https://www.blackhat.com/)

***

#### **3. Billion Laughs Attack (Recursive XXE DoS)**

* **Attack Vector:** Attackers used recursive entity references to **overload memory usage**, leading to **denial-of-service (DoS)**.
* **Impact:** XML parsers crashed due to **excessive memory consumption**, affecting **web services and APIs**.
* **Lesson Learned:** Disabling **DTD processing** prevents recursive entity injection.
* **Source:** [CVE-2017-9233 (IBM XML Parser DoS)](https://nvd.nist.gov/vuln/detail/CVE-2017-9233)

***

### **Attack Scenarios**

#### **1. Local File Disclosure via XXE**

**Attack URL:**

```xml
<!DOCTYPE foo [
    <!ENTITY exfil SYSTEM "file:///etc/passwd">
]>
<root>&exfil;</root>
```

**Explanation:**

* If an application **parses XML without proper security**, it may expose **internal system files** such as:
  * `/etc/passwd` (Linux user data).
  * `C:\Windows\win.ini` (Windows system configuration).

***

#### **2. Server-Side Request Forgery (SSRF) via XXE**

**Attack Payload:**

```xml
<!DOCTYPE foo [
    <!ENTITY ssrf SYSTEM "http://internal-admin-panel.local/">
]>
<root>&ssrf;</root>
```

**Explanation:**

* The attacker forces the server to **make requests to internal systems** (e.g., **admin panels, APIs**).
* This can be used to **scan internal networks** or **leak sensitive endpoints**.

***

### **Mitigation Strategies**

#### **1. Disable External Entity Processing**

**Secure Example (Python – defusedxml)**

```python
import defusedxml.ElementTree as ET
ET.parse('data.xml')  # Secure parsing without external entities
```

***

#### **2. Secure XML Parsing in Java**

**Example (Java – Secure XML Processing)**

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

***

#### **3. Validate and Sanitize Input**

* **Reject untrusted XML documents**.
* **Remove unnecessary features** such as **DTD and external entity resolution**.

**Example (PHP – Secure XML Parsing)**

```php
libxml_disable_entity_loader(true);
$xml = new DOMDocument();
$xml->loadXML($user_input, LIBXML_NOENT | LIBXML_DTDLOAD);
```

***

#### **4. Use JSON Instead of XML**

* Many modern applications **replace XML with JSON**, which **reduces the risk of XXE attacks**.

***

#### **5. Implement Web Application Firewalls (WAFs)**

* **Deploy security rules** to detect and block **XXE payloads**.
* **Monitor logs** for **unexpected XML parsing errors**.

***

### **Final Thoughts**

**XML External Entity (XXE) vulnerabilities** pose a **significant risk** to web applications by enabling:

* **Unauthorized data access** (e.g., reading `/etc/passwd`).
* **SSRF attacks** (e.g., querying internal resources).
* **Denial-of-Service (DoS) via recursive entity injection**.

#### **Key Takeaways:**

1. **Disable external entity resolution** in **all XML parsers**.
2. **Sanitize and validate XML input** before processing.
3. **Use secure XML libraries** such as **defusedxml (Python)** and **DocumentBuilderFactory (Java)**.
4. **Consider switching to JSON**, which **does not support entities**.

By following these **best practices**, organizations can **significantly reduce** the risk of XXE attacks.

***

### **Further Reading & References**

* [OWASP XXE Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
* [PortSwigger XXE Labs](https://portswigger.net/web-security/xxe)
* [CVE-2017-9233 – XXE in IBM XML Parser](https://nvd.nist.gov/vuln/detail/CVE-2017-9233)
* [HackerOne Uber XXE Report](https://hackerone.com/reports/408693)
