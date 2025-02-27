# Brute Forcing Authentication (PortSwigger Lab)

***

## **Lab - Brute Forcing Authentication**

### **Objective**

Use brute force techniques to identify valid username and password combinations.

[**Guided Lab: Brute Force Attacks**](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

### **Lab Steps**

#### 1. **Initial Login Attempt**

* Go to "My Account" and attempt a login with invalid credentials.
* Observe the error message (**"Invalid Username"**).
* Note: Username and password fields are both **8 characters long**.

#### 2. **Analyze the POST Request**

* Intercept the login request using **Burp Suite**.
*   Look for the `POST /login` request and examine the body:

    ```plaintext
    username=test&password=test
    ```

#### 3. **Username Enumeration with Burp Intruder**

* Add **Payload Position Marker** around the `username` field.
* Load a username wordlist (e.g., `/usr/share/seclists/Usernames/`).
* Start the attack and look for:
  * **High content length** (e.g., `3250`).
* Validate the username via response:
  * Check the **Render** tab for error messages like **"Incorrect Password."**

#### 4. **Password Brute Force**

* Repeat the process for the `password` field.
* Look for a significantly different **content length** (e.g., `187` vs. `3250`).

#### 5. **FUZZ with ffuf**

*   Save the POST request in a file:

    ```plaintext
    username=FUZZUSER&password=FUZZPASS
    ```
*   Run `ffuf` in **clusterbomb mode**:

    ```bash
    ffuf -request req.txt -request-proto https -mode clusterbomb \
    -w /home/kali/usernames.txt:FUZZUSER \
    -w /home/kali/passwords.txt:FUZZPASS -mc 302
    ```
* Results:
  * **Username:** `adkit`
  * **Password:** `michelle`

***

### **Observations**

* **Content Length** is the primary indicator of success (**e.g., `3250` vs. `187`**).
* Use **negative search filters** in Burp Suite for efficiency:
  * Filter out results with **"Invalid Username."**

***

### **Tools Used**

* **Burp Suite**
* **ffuf**

***
