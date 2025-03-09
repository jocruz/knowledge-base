# Mass Assignment Overview

## **Mass Assignment Vulnerability**

### **High-Level Summary**

**Mass Assignment** is a security vulnerability that arises when **web frameworks automatically bind input parameters to object properties** without sufficient validation. This feature is designed to simplify development, but when improperly implemented, it can lead to **unauthorized access** and **privilege escalation** by allowing attackers to modify sensitive fields.

#### **Key Insights**

* Many web frameworks (**Ruby on Rails, Laravel, Node.js, Django**) support **mass assignment features** to streamline data handling.
* If attackers discover sensitive fields (e.g., `"admin": true`), they can **modify privileges** by injecting values into **unsanitized requests**.
* Applications that expose hidden attributes in API responses may be vulnerable.

***

### **Technical Definitions**

| **Term**                    | **Definition**                                                                                       |
| --------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Mass Assignment**         | The automatic mapping of input data to object properties without explicit validation.                |
| **Object Binding**          | Automatically assigning request parameters to a backend model or database object.                    |
| **Spread Operator (`...`)** | A JavaScript feature (`const user = {...req.body}`) that merges user-provided data directly.         |
| **Privilege Escalation**    | Unauthorized modification of user privileges (e.g., changing `"role": "user"` to `"role": "admin"`). |
| **Fuzzing**                 | The process of injecting unexpected input values to test for vulnerabilities.                        |
| **Leaky API Endpoints**     | API responses that expose excessive or sensitive data that could be manipulated.                     |

***

### **Understanding Mass Assignment**

#### **1. Overview of Mass Assignment**

Mass Assignment is a **feature** in many web frameworks that allows **input parameters from HTTP requests to be automatically mapped to object attributes**. While this simplifies coding and reduces boilerplate, it can introduce security risks when **sensitive fields are unintentionally exposed**.

**Common frameworks that support mass assignment:**

* **Ruby on Rails** (`Active Record mass assignment`)
* **Laravel** (`$fillable` vs. `$guarded`)
* **Node.js** (`req.body` parsing via `express.json()`)
* **Django** (`ModelForms` and `serializers`)

***

### **How Mass Assignment Works**

#### **Safe Implementation**

**Example: Correctly Implemented Input Validation**

```javascript
// Incoming POST Request Data
"user": {
   "firstName": "John",
   "middleName": "Michael",
   "lastName": "Doe"
}

// Node.js Code (Whitelisted Fields)
const allowedFields = ['firstName', 'middleName', 'lastName'];
const user = {};

allowedFields.forEach(field => {
    if (req.body[field]) {
        user[field] = req.body[field];
    }
});

console.log(user);
// Output: John Michael Doe
```

**Explanation**

* Only explicitly defined fields (`firstName`, `middleName`, `lastName`) are assigned.
* Additional fields such as `"admin": true` are **ignored**.

***

#### **Privilege Escalation via Mass Assignment**

**Example: Vulnerable Code (Privilege Escalation Possible)**

```javascript
// Incoming POST Request Data (Modified by Attacker)
"user": {
   "firstName": "Alice",
   "lastName": "Smith",
   "admin": true
}

// Node.js Code (Vulnerable Implementation)
const user = { ...req.body };

// Output: Alice Smith, Admin: true (Unauthorized Privilege Escalation)
```

**How It Works**

* The attacker **injects `"admin": true`** into the request.
* The server **blindly binds** all request parameters to the user object.
* The attacker **gains admin privileges** without proper authorization.

***

### **Identifying Mass Assignment Vulnerabilities**

#### **Key Techniques for Discovery**

1. **Fuzzing API Requests**
   * Inject parameters such as `"admin": true`, `"role": "superuser"`, or `"balance": 9999999"`.
   * Observe whether unauthorized modifications occur.
2. **Inspect API Responses**
   * If responses contain hidden fields (e.g., `"role": "admin"`), these may be **modifiable via mass assignment**.
3. **JWT Token Analysis**
   * Decode JWT tokens to check for modifiable claims.
4. **Source Code Review**
   * Identify mass object binding functions (`...req.body` in Node.js).
5. **Testing with Burp Suite**
   * Intercept requests and modify JSON fields to assess unauthorized field acceptance.

***

### **Real-World Incidents of Mass Assignment**

#### **1. Uber (2016)**

* **Vulnerability:** Attackers exploited mass assignment in Uber’s API to **modify driver payout amounts**.
* **Impact:** Allowed unauthorized users to change fare rates, leading to financial losses.
* **Reference:** [HackerOne Uber Report](https://hackerone.com/reports/218748)

#### **2. GitHub Enterprise (2020)**

* **Vulnerability:** Attackers found that GitHub’s API exposed an undocumented `role` parameter.
* **Impact:** A non-admin user could elevate privileges by sending `"role": "admin"`.
* **Reference:** [CVE-2020-10526](https://nvd.nist.gov/vuln/detail/CVE-2020-10526)

#### **3. Shopify (2022)**

* **Vulnerability:** Researchers found mass assignment issues in GraphQL APIs.
* **Impact:** Unauthorized changes to **discount rules** and **product prices**.
* **Reference:** [CVE-2022-23764](https://nvd.nist.gov/vuln/detail/CVE-2022-23764)

***

### **Exploitation Process (Step by Step)**

#### **1. Identify Input Fields**

* Review `POST`/`PUT` requests and inspect data models for vulnerable attributes.

#### **2. Modify the Request**

*   Inject fields such as:

    ```json
    {
      "role": "admin",
      "balance": "9999999"
    }
    ```

#### **3. Submit the Modified Request**

* Use **Burp Suite** or **Postman** to submit modified requests.

#### **4. Observe the Response**

* If unauthorized modifications occur, the application is vulnerable.

***

### **Defenses Against Mass Assignment**

#### **1. Use Whitelisting for Allowed Fields**

Explicitly define which input fields can be modified.

```javascript
const allowedFields = ['firstName', 'middleName', 'lastName'];
const user = {};

allowedFields.forEach(field => {
    if (req.body[field]) {
        user[field] = req.body[field];
    }
});
```

#### **2. Restrict Writable Attributes in ORM**

If using **Sequelize (Node.js ORM)**, restrict which attributes are assignable:

```javascript
const User = sequelize.define("User", {
    firstName: { type: DataTypes.STRING, allowNull: false },
    admin: { type: DataTypes.BOOLEAN, allowNull: false, defaultValue: false }
}, {
    defaultScope: {
        attributes: { exclude: ['admin'] }
    }
});
```

#### **3. Implement Role-Based Access Control (RBAC)**

Ensure only authorized users can modify sensitive fields.

#### **4. Sanitize User Input**

Use libraries such as **Express Validator** to validate user input.

```javascript
app.post('/update', [
    body('role').not().exists(), // Prevents unauthorized role modification
    body('email').isEmail()
], (req, res) => {
    // Proceed only if valid
});
```

#### **5. Monitor API Activity**

* Log **unusual account modifications**.
* Implement **rate-limiting** for sensitive API endpoints.

***

### **Penetration Testing Checklist for Mass Assignment**

* [ ] **Test API Endpoints:** Inject unexpected fields and observe behavior.
* [ ] **Inspect JWT Tokens:** Decode and check for sensitive claims.
* [ ] **Scan API Responses:** Look for sensitive attributes in API responses.
* [ ] **Check Source Code:** Identify object binding patterns (`...req.body`).
* [ ] **Validate Form Fields:** Ensure sensitive fields are **restricted server-side**.

***

### **Additional Resources**

* [OWASP Mass Assignment Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
* [JWT.IO Token Analyzer](https://jwt.io/)

***

### **Final Takeaways**

* **Mass Assignment** can lead to **privilege escalation and unauthorized modifications**.
* Always **whitelist and validate user input**.
* Perform **regular API testing** to detect **hidden and vulnerable fields**.
