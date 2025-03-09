# JWT\_Tool Overview

## JWT\_Tool – Cracking and Manipulating JSON Web Tokens (JWTs)

### High-Level Summary

**JWT\_Tool** is a Python-based security testing tool designed for analyzing the security of **JSON Web Tokens (JWTs)**. It supports **decoding, cracking, modifying, and exploiting** JWTs, primarily targeting weak **HMAC-SHA tokens**.

**Purpose:**

* **Crack JWT secrets** when weak HMAC keys are used.
* **Test for insecure JWT configurations**, such as missing expiration claims or poorly implemented signature verification.
* **Modify JWT payloads** after obtaining the secret.

***

### Definition Bank

| **Term**                   | **Definition**                                                                                |
| -------------------------- | --------------------------------------------------------------------------------------------- |
| **JWT\_Tool**              | A Python tool for security testing of JWTs, including cracking weak HMAC secrets.             |
| **HMAC-SHA256**            | A symmetric algorithm where the same key is used for both **signing** and **verifying** JWTs. |
| **`--crack` Flag**         | Instructs JWT\_Tool to attempt brute-forcing the JWT's secret using a password list.          |
| **`-d` Flag (Dictionary)** | Specifies a wordlist for brute-force attempts against JWT secrets.                            |
| **Xato-1000 Wordlist**     | A common password list in `/usr/share/seclists`, useful for testing weak secrets.             |

***

### Key Features and Usage of JWT\_Tool

#### 1. Cracking HMAC-SHA256 JWTs (`--crack` Flag)

JWTs signed using **HMAC-SHA256** can be **brute-forced** if they rely on weak secrets.

**Basic Syntax:**

```bash
python3 jwt_tool.py [JWT] --crack -d [wordlist]
```

**Example:**

```bash
python3 jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJsb2dpbiI6InRlc3QifQ.aqNCvShlNT9jBFTPBpHDbt2gBB1MyHiisSDdp8SQvgw --crack -d /usr/share/seclists/SecLists-master/Passwords/xato-net-10-million-passwords-1000.txt
```

**Expected Output (Successful Crack):**

```
[*] Attempting to crack HMAC key...
[+] Key Found: secret123
```

If a secret key is found, attackers can **forge JWTs** and escalate privileges.

***

#### 2. Using the `-d` Flag to Specify a Wordlist

The `-d` flag allows specifying a custom **password list** for brute-force attempts.

**Example Path to Common Wordlist (Kali Linux):**

```plaintext
/usr/share/seclists/SecLists-master/Passwords/xato-net-10-million-passwords-1000.txt
```

**Why This Wordlist?**

* Contains **common weak passwords**.
* Frequently used for **HMAC-SHA256 JWT cracking**.
* Often included in **default JWT implementations with weak secrets**.

***

#### 3. Modifying a Token After Cracking (JWT Tampering)

Once the secret key is obtained, a **JWT payload can be modified** and re-signed.

**Steps to Modify and Re-Sign a JWT:**

1. **Crack the JWT secret** using the `--crack` flag.
2. Use [**jwt.io**](https://jwt.io/) to decode the JWT.
3. Modify the **payload** (e.g., change `"user": "guest"` to `"user": "admin"`).
4. Re-sign the token using the cracked secret (`secret123`).
5. Use the new JWT to test privilege escalation.

***

### Common Attacks Using JWT\_Tool

| **Attack Type**                         | **Description**                                               | **How to Test**                                                    |
| --------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Weak Secret Cracking**                | JWTs using predictable secrets can be brute-forced.           | Run the `--crack` flag with a password list.                       |
| **Privilege Escalation**                | Modify `"role": "user"` to `"role": "admin"`.                 | Crack the secret, edit the payload, and re-sign the JWT.           |
| **Replay Attacks**                      | JWTs without expiration can be reused indefinitely.           | Attempt using an old JWT after logout.                             |
| **Algorithm Confusion (`None` Attack)** | If the server allows `alg: none`, signatures can be bypassed. | Modify the JWT header to `"alg": "none"` and remove the signature. |

***

### Preventing JWT Vulnerabilities

* **Use Strong Secrets:** Ensure secrets are **long and randomly generated**.
* **Disable Symmetric Algorithms (HS256):** Prefer **RS256** to prevent key brute-forcing.
* **Enforce Expiration (`exp` Claim):** Expired tokens should be **rejected by the server**.
* **Restrict Token Scope:** Minimize the amount of data stored in JWT payloads.
* **Do Not Hardcode Secrets:** Secrets should be stored securely and rotated periodically.

***

### Secure JWT Example

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
.
{
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 1714044800
}
.
[Signature]
```

**Why Secure?**

* Uses **asymmetric encryption (RS256)** instead of **HMAC**.
* Includes an **expiration claim (`exp`)**.
* Stores **minimal sensitive data** in the payload.

***

### Installing JWT\_Tool in Kali Linux

```bash
git clone https://github.com/ticarpi/jwt_tool.git /opt/jwt_tool
cd /opt/jwt_tool
pip3 install -r requirements.txt
```

***

### Penetration Testing Workflow for JWTs

1. **Capture a JWT** from an authenticated session.
2. **Decode it using jwt.io** to examine header, payload, and algorithm.
3.  **Test for Weak Secrets** using JWT\_Tool:

    ```bash
    python3 jwt_tool.py [JWT] --crack -d /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000.txt
    ```
4. **Modify the Token (If Cracked):**
   * Change **user role, expiration, or claims**.
   * Re-sign with the cracked key.
5. **Replay or Send Modified JWT** to test **authentication and privilege escalation**.

***

### Final Takeaways

* **JWT\_Tool** is highly effective for **brute-forcing weak JWT secrets**.
* **Xato-1000 password list** provides a good baseline for testing **HMAC weaknesses**.
* **Asymmetric JWTs (RS256) should be used** instead of HMAC for security.
* **JWT-based authentication should implement expiration checks** to prevent token reuse.

By following best practices and leveraging **secure JWT configurations**, applications can significantly reduce the risk of **token forgery and privilege escalation attacks**.
