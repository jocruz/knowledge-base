# 2FA broken logic (Portswigger Lab)

You're right—I’ll make sure all your original steps are included while keeping everything professional and structured for an employer-facing GitBook page. Here’s the full version with **Steps 6, 7, and 8** properly incorporated:

***

#### **Exploiting Broken Logic in 2FA – PortSwigger Lab Analysis**

**Lab Description**

This lab demonstrates a **broken logic vulnerability** in a multi-factor authentication (MFA) implementation, where an attacker can manipulate the `verify` parameter to generate a 2FA code for another user. The lab’s goal is to bypass 2FA and access **Carlos’s account** without knowing his MFA code.

* **Lab URL**: [PortSwigger 2FA Broken Logic Lab](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
* **Credentials**:
  * Attacker: `wiener:peter`
  * Victim: `carlos:montoya`

***

### **Analysis and Exploitation**

#### **1. Understanding the 2FA Process**

By logging in with the attacker’s credentials (`wiener:peter`), it was possible to analyze how the application handles MFA. The authentication flow included:

* **Step 1: Initial Login (`POST /login`)**
  * Sends the **username and password**.
  * Issues a **session cookie (`session`)**.
* **Step 2: 2FA Code Request (`GET /login2`)**
  * The request includes a `verify` parameter tied to the logged-in user.
  * A 4-digit MFA code is generated and sent to the user.

#### **2. Identifying the Vulnerability**

Using **Burp Suite**, the `verify` parameter was found to **control which user receives an MFA code**. This introduced a critical security flaw:

* Modifying the `verify` parameter in the `GET /login2` request from `wiener` to `carlos` resulted in generating a **new MFA code for Carlos**.
* The MFA code was **not session-bound**, meaning an attacker could request it without Carlos logging in.

#### **3. Exploiting the Vulnerability**

**Step 1: Log In to Your Account**

1. Log in with **wiener:peter** and complete the 2FA step.
2. Intercept the login process in Burp Suite and inspect:
   * The `POST /login` request.
   * The `GET /login2` request, which includes the `verify` parameter.

**Step 2: Investigate the 2FA Process**

1. Observe the cookies in the intercepted requests:
   * **Example**: `Cookie: verify=wiener; session=rQJbtZy0DedQsXg0psIKs88QIMboLft`.
   * Note how the `verify` cookie corresponds to the username.
2. Identify the `mfa-code` parameter (e.g., `mfa-code=0962`) in the `/login2` request.

**Step 3: Generate Carlos’s MFA Code**

1. Send the `GET /login2` request to **Repeater**.
2. Modify the `verify` parameter from `wiener` to `carlos`.
3. Send the modified request. This action generates an MFA code for Carlos.

**Step 4: Brute Force Carlos’s MFA Code**

1. Locate the `POST /login2` request and send it to **Intruder**.
2. Configure payloads:
   * Set `verify=carlos`.
   * Place a payload position on the `mfa-code` parameter.
3. Use a **numeric brute-force attack** (0000-9999) to find the correct MFA code.
4. Identify the **successful response (302 redirect)** that confirms authentication.

<figure><img src="../../../.gitbook/assets/Pasted image 20241209155307.png" alt=""><figcaption></figcaption></figure>



**Step 5: Identify the Valid MFA Code**

1. Start the **Intruder** attack.
2. Look for a response with a **302 status code**, indicating a successful login.
3. Identify the **valid MFA code** (e.g., `0262`).

**Step 6: Capture the Session Cookie**

1. Switch to the **Response** tab in Burp Suite for the successful request.
2. Copy the **session cookie** issued after brute-forcing Carlos’s MFA code.

**Step 7: Inject the Cookie into the Browser**

1. Open the **browser developer tools** and go to the **Application tab**.
2. Replace the existing session cookie with the one obtained from Burp Suite:
   * Delete the existing `verify=wiener` and `session` cookies.
   * Inject the captured session cookie corresponding to Carlos.
3. Save the changes.

<figure><img src="../../../.gitbook/assets/Pasted image 20241209155123.png" alt=""><figcaption></figcaption></figure>



**Step 8: Access Carlos’s Account**

1. Navigate to `/my-account?id=carlos` in the browser.
2. Confirm that you have successfully accessed Carlos’s account page.

<figure><img src="../../../.gitbook/assets/Pasted image 20241209155147.png" alt=""><figcaption></figcaption></figure>



***

### **Why This Works**

This vulnerability arises because the **MFA codes are not tied to individual sessions**:

* The application **incorrectly allows attackers to generate MFA codes for any user** by modifying the `verify` parameter.
* **Brute-forcing** the 4-digit MFA code is feasible due to its **limited range (0000-9999)**.
* The application **does not enforce rate-limiting**, making brute-force attacks trivial.

***

### **Key Takeaways**

* **MFA codes must be session-bound** to prevent unauthorized users from generating codes.
* **Rate-limiting and account lockouts** should be enforced to prevent brute-force attacks.
* **User authentication should be properly validated** before issuing MFA codes.

***

### **Security Best Practices to Prevent This Flaw**

#### **1. Strengthen MFA Token Generation**

* MFA codes should be tied to both the **user’s session and device**.
* **Do not allow arbitrary modification of user-related authentication parameters** (e.g., `verify`).

#### **2. Implement Brute-Force Protections**

* Enforce **rate-limiting** and introduce a **lockout mechanism** for multiple failed MFA attempts.
* Implement **time-based expiry** for MFA codes.

#### **3. Restrict Session Hijacking Opportunities**

* **Invalidate previous session cookies** after a successful MFA verification.
* Ensure that session management **validates MFA completion** before allowing access to sensitive areas.

***

### **Final Thoughts**

This lab highlights how a **misconfigured MFA implementation** can introduce severe security risks. Without properly enforcing session-based authentication, an attacker can **manipulate parameters**, **brute-force MFA codes**, and **hijack user sessions**. Identifying these logic flaws is critical when assessing authentication security, ensuring that multi-factor authentication serves its intended purpose rather than becoming a security liability.

