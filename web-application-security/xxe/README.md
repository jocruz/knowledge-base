# XXE

## XML External Entity (XXE) Vulnerabilities

### **Overview**

**Extensible Markup Language (XML)** is widely used for data exchange between applications due to its flexibility and readability. However, XML parsers often support **external entities**, which, if not properly secured, can be exploited through **XML External Entity (XXE) attacks**. These attacks allow an attacker to:

* Access sensitive **local files** on the server.
* Execute **server-side request forgery (SSRF)** attacks.
* Leak **environment variables and internal configurations**.
* Perform **denial-of-service (DoS) attacks** through recursive entity expansion (billion laughs attack).

**Understanding XXE vulnerabilities is critical because:**

* **Risk:** Can lead to **data breaches**, **server compromise**, and **privilege escalation**.
* **Impact:** Unauthorized access to **system files**, **internal services**, and **sensitive configurations**.
* **Mitigation:** Helps developers implement **secure XML processing** to protect web applications.

***

### **Definition Bank**

| **Term**                             | **Definition**                                                            |
| ------------------------------------ | ------------------------------------------------------------------------- |
| **XML (Extensible Markup Language)** | A markup language used for structuring and transporting data.             |
| **DTD (Document Type Definition)**   | A set of rules defining the structure and types of data in an XML file.   |
| **Entity**                           | A reusable reference in an XML document, defined in the DTD.              |
| **External Entity**                  | An entity defined in a DTD that references an external resource or file.  |
| **XXE (XML External Entity Attack)** | A vulnerability where an attacker can inject malicious external entities. |

***

### **Key Concepts and Explanations**

#### **1. How XML Works**

XML follows a **tree-structured format**, allowing data to be represented in a structured and hierarchical manner.

**Example of a Simple XML Document:**

```xml
<user>
    <username>admin</username>
    <email>admin@example.com</email>
</user>
```

***

#### **2. What is an External Entity?**

XML allows the definition of **entities**, which act as placeholders for data. **External entities** reference an external file or resource, making them vulnerable if not properly controlled.

**Example of a Local Entity Definition:**

```xml
<!DOCTYPE foo [
    <!ENTITY example "Hello, World!">
]>
<root>&example;</root>
```

**Exploitable External Entity Example:**

```xml
<!DOCTYPE foo [
    <!ENTITY exfil SYSTEM "file:///etc/passwd">
]>
<root>&exfil;</root>
```

**Why This is Dangerous:**

* The external entity `exfil` references a **local file** (`/etc/passwd`).
* If the XML parser processes this request, it **retrieves and discloses** the file's contents.

***

#### **3. How XXE Attacks Work**

1. The attacker injects **a malicious external entity** into an XML document.
2. The XML parser processes the entity and **fetches sensitive data**.
3. The attacker retrieves the leaked data through **error messages, responses, or blind exfiltration**.

***

### **Real-World XXE Exploits**

#### **1. Uber Bug Bounty Disclosure (2019)**

* A security researcher exploited an **XXE vulnerability** to retrieve AWS metadata, leading to **server-side request forgery (SSRF)** and internal resource access.
* **Impact:** Gained access to **internal services**.

#### **2. Samsung SmartThings XXE Attack**

* Attackers exploited **XXE in XML-based APIs**, leaking **sensitive configurations**.
* **Impact:** Unauthorized disclosure of **user credentials and device data**.

#### **3. Billion Laughs Attack (Recursive XXE DoS)**

* Attackers crafted **recursive entity references** to crash XML parsers.
* **Impact:** **Denial-of-service (DoS)** due to excessive memory consumption.

***

### **Mitigation Strategies**

#### &#x20;**Best Practices for Preventing XXE**

| **Approach**                 | **Secure?** | **Explanation**                                                 |
| ---------------------------- | ----------- | --------------------------------------------------------------- |
| **Disabling DTD Parsing**    | ✅           | Prevents use of external entities altogether.                   |
| **Using Secure XML Parsers** | ✅           | Restricts external entity resolution.                           |
| **Sanitizing XML Input**     | ✅           | Filters user-controlled input to prevent XML injection.         |
| **Switching to JSON**        | ✅           | JSON lacks DTDs and external entities, reducing attack surface. |

#### **Example of Secure XML Parsing (Python)**

```python
import defusedxml.ElementTree as ET
ET.parse('data.xml')  # Secure parsing without external entities
```

#### **Example of Secure XML Parsing (Java)**

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

***

### **Conclusion**

**XML External Entity (XXE) vulnerabilities** pose a significant threat to web applications by enabling **unauthorized data access**, **server-side request forgery (SSRF)**, and **denial-of-service attacks**. Organizations must implement **secure XML parsing techniques**, **disable external entity resolution**, and **regularly conduct security assessments** to mitigate these risks effectively.
