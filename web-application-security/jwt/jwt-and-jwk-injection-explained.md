# JWT and JWK Injection Explained

## JWT and JWK Injection Explained

### What is a JWT (JSON Web Token)?

A **JWT** is a token used to **prove a user's identity** in web applications. It consists of three parts:

```
Header.Payload.Signature
```

#### **1. Header**

* Specifies the **algorithm** used for signing.
* Example:

```json
{
  "alg": "RS256"
}
```

#### **2. Payload**

* Contains **claims** (data) about the user.
* Example:

```json
{
  "sub": "wiener",
  "role": "user"
}
```

#### **3. Signature**

* A **cryptographic signature** is generated using:
  * The **server’s private key**.
  * The **Header + Payload** combined.

Example of how the signature is computed:

```
Signature = sign(Header + Payload, privateKey)
```

* This ensures the token **wasn't tampered with**.

***

### How JWT Verification Works (Secure Process)

When a server **receives a JWT**, the verification process follows these steps:

1. **Splits the Token**
   * Breaks it into `Header.Payload.Signature`.
2. **Ignores Any Key Inside the Token**
   * The server **should not trust any key provided in the JWT itself**.
3. **Recalculates the Signature**
   * Using its **trusted public key**.
4. **Compares Signatures**
   * If the **computed signature** matches the **one in the token**, the token is **valid** ✅.
   * If they **don’t match**, the token is **rejected** ❌.

#### **Example Code (Secure Verification)**

```javascript
const jwt = require('jsonwebtoken');
const fs = require('fs');

const publicKey = fs.readFileSync('./public.pem');

const token = "eyHeader.eyPayload.Signature";

// Verify the token using the **trusted** public key
jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

***

### What is a JWK (JSON Web Key)?

A **JWK** is a public key formatted in **JSON**.

#### **Example JWK**

```json
{
  "kty": "RSA",
  "n": "attacker-controlled-public-key",
  "e": "AQAB"
}
```

#### **Legitimate Use of JWKs**

JWKs are commonly used when services need a **public key rotation mechanism**. Some examples include:

* Google and Microsoft providing public keys for **OAuth authentication**.
* Applications fetching trusted keys from a **secure key URL**.

***

### How Does JWK Injection Work?

JWK injection occurs when a **developer mistakenly allows a JWT to include its own public key** inside the token.

#### **Exploit Steps**

#### **1. Attacker Creates a JWT with a Fake Key**

The attacker **modifies the JWT header** to include a **malicious JWK**.

```json
{
  "alg": "RS256",
  "jwk": {
     "kty": "RSA",
     "n": "attacker-public-key",
     "e": "AQAB"
  }
}
```

#### **2. Attacker Signs the Token with Their Private Key**

* The attacker generates **their own RSA key pair**.
* They **sign the JWT** using their **private key**.

#### **3. Vulnerable Server Code**

Some applications improperly **extract and trust the key inside the JWT** instead of using a pre-approved key.

```javascript
const decoded = jwt.decode(token, { complete: true });
const jwk = decoded.header.jwk;  // 🚨 Attacker injects a fake key here

// 🚨 Server trusts the attacker's key instead of its own stored key
jwt.verify(token, jwk.n, { algorithms: ['RS256'] });
```

#### **4. Why This Works**

* The attacker **controls the entire key pair**.
* They sign the JWT with their **private key**.
* The server **trusts the key provided inside the JWT header**.
* Since the attacker **controls both the public key (JWK) and signature**, the token appears **valid**.

***

### Proper Fix: Secure JWT Verification

To **prevent JWK injection**, the server should **always** verify JWTs using a **trusted key from its own storage**.

#### **Correct Code**

```javascript
const publicKey = fs.readFileSync('./public.pem');
jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

#### **Key Differences**

| **Correct Approach** ✅                                             | **Vulnerable Approach** ❌                                      |
| ------------------------------------------------------------------ | -------------------------------------------------------------- |
| Uses **pre-approved** public keys for verification.                | Allows JWT to specify its **own key**.                         |
| The public key is **hardcoded or retrieved from a trusted store**. | The public key is **provided by the attacker** inside the JWT. |

***

### Why Do Developers Make This Mistake?

1. **Misunderstanding Key Rotation**
   * Some developers believe allowing JWTs to include their own keys helps with **key rotation**, but it **defeats the purpose of signed JWTs**.
2. **Misconception About JWT Verification**
   * Some assume JWTs should "self-validate" instead of **verifying against a trusted public key**.
3. **Ease of Implementation**
   * Accepting JWKs from the JWT itself **simplifies distributed authentication**, but it breaks **integrity**.

***

### Final Recap

* **JWTs** are signed by a **trusted server** using a **private key**.
* **Servers must verify JWTs using their own trusted public key**.
* **JWK Injection** occurs when a **server trusts a public key inside the token** instead of **validating it against a known source**.
* **Attackers can generate valid tokens** if they can inject their own key.

***

### Key Takeaways

| ✅ **Best Practices**                                          | ❌ **Common Mistakes**                               |
| ------------------------------------------------------------- | --------------------------------------------------- |
| Always verify JWTs using **trusted public keys**.             | Trusting public keys **inside** the JWT itself.     |
| Disable `jwk` injection in JWT verification libraries.        | Fetching **unverified keys** from external sources. |
| Use **pre-configured** key stores or key management services. | Assuming JWTs should "self-validate."               |

***

### Additional Resources

* [JWT.io Token Editor & Decoder](https://jwt.io/)
* [RFC 7517 - JSON Web Key (JWK) Specification](https://datatracker.ietf.org/doc/html/rfc7517)

By following **proper JWT validation practices**, organizations can **prevent JWK injection attacks** and **ensure token integrity**.
