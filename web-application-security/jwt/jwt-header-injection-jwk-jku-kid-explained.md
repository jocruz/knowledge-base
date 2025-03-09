# JWT Header Injection (JWK, JKU, KID Explained)

## JWT Header Injection (JWK, JKU, KID Explained)

### High-Level Summary

**JWT Header Injection Attacks** exploit improperly validated **JWT headers** to manipulate the server’s **key verification process**. By modifying headers like **JWK, JKU, and KID**, attackers can potentially **bypass signature validation** and forge JWTs.

**Key Insight:**

* If the server does not **properly validate the JWT headers**, an attacker may be able to **inject a public key** or redirect the server to an external key they control.
* This allows attackers to **forge authentication tokens** and **gain unauthorized access**.

***

### Definition Bank

| **Term**                   | **Definition**                                                                                                                    |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **JWT Header Injection**   | An attack that manipulates JWT headers to control key verification and potentially bypass security mechanisms.                    |
| **JWK (JSON Web Key)**     | A **JSON object** that contains a **public key**, which can be embedded in the JWT header.                                        |
| **JKU (JSON Web Key URL)** | A **URL** that references a **JSON Web Key Set (JWKS)**, allowing the server to retrieve keys dynamically.                        |
| **KID (Key ID)**           | A **Key Identifier** used when multiple signing keys exist, helping the server select the correct key for signature verification. |

***

### Key Concepts and Explanation

#### 1. What is JWT Header Injection?

* JWT Header Injection exploits the **JWK, JKU, and KID fields** in the **JWT header** to **bypass verification checks**.
* The attacker **tricks the server** into accepting an **untrusted** or **malicious** key for verifying token signatures.

✅ **Primary Goal:** Trick the server into using a **key controlled by the attacker** instead of a trusted key.

***

#### 2. JWK (JSON Web Key) Injection

The **JWK header** allows an attacker to **embed a public key** inside the JWT itself.

✅ **Expected Secure Behavior:**

* The server should **reject unknown JWK keys** and verify the key against a **trusted key store**.

🚨 **Exploitation:**

* The attacker **injects their own public key** into the JWT header.
* If the server **trusts embedded keys** without verifying them against a trusted list, the attacker can **generate valid tokens**.

**Example: Malicious JWT Header**

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "jwk": {
    "kty": "RSA",
    "n": "MIIBIjANBgkqhkiG9w...",
    "e": "AQAB"
  }
}
```

✅ **Security Concern:**

* If the server **trusts the JWK in the token** without cross-checking against a known key store, an attacker can sign their own JWTs.

***

#### 3. JKU (JSON Web Key URL) Injection

The **JKU (JSON Web Key URL)** field specifies a **URL** from which the server retrieves the **public key**.

✅ **Expected Secure Behavior:**

* The server should **only accept JKU values** that point to **pre-approved trusted sources**.

🚨 **Exploitation:**

* The attacker **hosts their own malicious JWKS** at a URL they control.
* The server fetches the **attacker-controlled key** and **accepts it as valid**.

**Example: Malicious JWT Header**

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "jku": "https://evil.com/malicious-jwks.json"
}
```

✅ **Security Concern:**

* If the server does **not validate the JKU URL**, it may download and use a **malicious signing key**, allowing attackers to **forge JWTs**.

***

#### 4. KID (Key ID) Manipulation

The **KID (Key ID)** field tells the server which **key** to use for verification.

✅ **Expected Secure Behavior:**

* The server should only accept **valid, pre-registered KIDs**.

🚨 **Exploitation:**

* Attackers modify the `KID` value to:
  1. **Point to an invalid or weak key**, making the server default to **no verification**.
  2. **Inject a SQL or command injection payload**, attempting to exploit weak key selection logic.

**Example: Bypassing Signature Verification**

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "none"
}
```

✅ **Security Concern:**

* If the server **accepts arbitrary KIDs** or **fails open** when the key is not found, the attacker may be able to **bypass authentication**.

***

### How Header Injection Works (Exploit Process)

1. **Inspect the Token:** Extract the **JWT header** and check for `JWK`, `JKU`, or `KID` fields.
2. **Modify the Header:** Inject a **malicious key** or redirect to a **controlled key URL**.
3. **Re-sign the Token:** Use tools like **jwt.io** to modify and re-sign the token.
4. **Send the Token:** Submit the modified JWT to the server.
5. **Observe:** If the server **accepts the forged token**, it is **not properly validating the key**.

***

### Pen Testing Use Cases for Header Injection

| **Attack Type**      | **Objective**                                        | **How to Test**                                                         |
| -------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------- |
| **JWK Injection**    | Inject an attacker-controlled public key.            | Modify the JWT header and check if the server accepts the key.          |
| **JKU Injection**    | Point the server to an external malicious key store. | Modify the `JKU` value to a controlled URL and observe server behavior. |
| **KID Manipulation** | Use an invalid or weak `KID` to bypass validation.   | Modify `KID` and test if the server still accepts the token.            |

***

### Defenses Against Header Injection Attacks

✅ **Key Validation:** Always validate JWT keys against a **trusted key store**.\
✅ **Reject External URLs:** Block **JKU references** to untrusted sources.\
✅ **Disable Weak Algorithms:** Avoid using **HS256** with weak secrets.\
✅ **Use Asymmetric Signing (RS256/ES256):** Ensure key pairs are properly managed.\
✅ **Enforce KID Validation:** Ensure `KID` values match **valid key entries only**.\
✅ **Limit Accepted JWT Headers:** Block unnecessary header fields such as `JKU` unless absolutely required.

***

### Secure JWT Example (Proper Key Management)

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "trusted-key-id"
}
```

&#x20;**Why Secure?**

* Uses **asymmetric encryption** (`RS256`).
* **KID is validated** against **pre-approved trusted keys**.
* **No external JKU URLs or JWK injections**.

***

### Summary of Exploit Steps

1. **Identify Vulnerability:** Check if `JWK`, `JKU`, or `KID` fields are present.
2. **Inject a Malicious Key:**
   * **JWK**: Embed a custom key.
   * **JKU**: Point to a malicious key store.
   * **KID**: Attempt a bypass using an invalid identifier.
3. **Re-sign and Test the Token:**
   * Use **jwt.io** to modify and test.
4. **Confirm Bypass:** If the server **accepts the forged token**, the **key validation is flawed**.

***

### Additional Resources

* [JWT.io Token Editor & Decoder](https://jwt.io/)
* [RFC 7517 - JSON Web Key (JWK) Specification](https://datatracker.ietf.org/doc/html/rfc7517)

***

### Final Takeaways

* **JWT Header Injection** allows attackers to **manipulate key verification**.
* Servers must **validate JWT keys against a trusted store**.
* Avoid relying on **JKU URLs, embedded JWKs, or weak KID implementations**.
* Always enforce **strong key management policies** to **prevent token forgery**.

By applying **proper JWT validation** and **avoiding common implementation flaws**, developers can **mitigate risks associated with JWT Header Injection attacks**.
