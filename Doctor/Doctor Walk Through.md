
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

Alright we don't get anything reflected back, lets just quickly check the source code to see what's up!

![[source-code-xss.png]]

Looks like our payload is being filtered out, so this saves us some time in future XSS payloads, perhaps we will need to bypass their filer, but for now lets just keep this in our notes and move on to template injection!

I personally like template injection because if successful it could lead to vertical movement so if you are unfamiliar 