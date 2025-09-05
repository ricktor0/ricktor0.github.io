---
title: "Ignite | Tryhackme"
date: 2024-07-04 10:00:00 +0000
categories: [Walkthrough, TryHackMe]
tags: [fuelcms, rce, privilege escalation]
---

![](https://cdn-images-1.medium.com/max/800/1*K09FnF_bJQKBBbLbJZVk9A.png)
{: .w-100 }

[Ignite](https://tryhackme.com/r/room/ignite) is an easy machine on [Tryhackme](https://tryhackme.com/) that focuses on vulnerability discovery and exploitation in [FuelCMS](https://www.getfuelcms.com/), a popular open-source content management system (CMS). It is a PHP-based CMS that provides a flexible and modular framework for building custom web applications.

We’ll be performing basic enumeration and privilege escalation to solve this machine.

### **Task 1 User.txt**

First of all let’s use [Rustscan](https://github.com/RustScan/RustScan) and [Nmap](https://nmap.org/) for basic port scanning.

```bash
rustscan --ulimit 100000 -a 10.10.145.64 -- -sV -sC -v -oN ignite
```

![](https://cdn-images-1.medium.com/max/800/1*kHC74a4lau6IW0Tb3xYiZA.png)
{: .float-left }

I personally prefer rustscan because RustScan is significantly faster than Nmap, especially when scanning large networks or performing multiple scans simultaneously, you can use rustscan to identify open ports and then nmap for detailed scanning


![Rustscan output](https://cdn-images-1.medium.com/max/800/1*kHC74a4lau6IW0Tb3xYiZA.png)
{: .float-left .me-3 }

I personally prefer rustscan because RustScan is significantly faster than Nmap, especially when scanning large networks or performing multiple scans simultaneously. You can use rustscan to identify open ports and then use Nmap for detailed scanning.

**Result of the Nmap scan**

![Nmap scan results](https://cdn-images-1.medium.com/max/800/1*JeETCDbW4oHMDvP1HnpIDg.png)
{: .w-100 }

It says port 80 is open and running Fuel CMS on it. Let’s check what we can find there in a web browser:

![FuelCMS Default Page](https://cdn-images-1.medium.com/max/800/1*bNHIN_N4dHlROp_eOBM0Aw.png)
{: .w-100 }

We are welcomed with the [Fuelcms](https://www.getfuelcms.com/) default landing page. Fuel [CMS](https://www.getfuelcms.com/) is built on top of the CodeIgniter framework and uses a modular architecture, which allows developers to easily add or remove features and functionality as needed.

On the default page, some things caught my attention:

1.  **The version of the CMS.** We can search for potential exploits with this info later.
    ![FuelCMS Version](https://cdn-images-1.medium.com/max/800/1*C03JLbnuZZiJq7Ky0HxqHg.png)
    {: .float-left .me-3 }

2.  **DEFAULT CREDENTIALS!!**
    ![Default Credentials Info](https://cdn-images-1.medium.com/max/800/1*F92u_YJ8ca8Cv6TMlReQgg.png)
    {: .float-left .me-3 }

> The `/fuel` directory leads to the login form, and it can be accessed with default credentials.

![FuelCMS Login Page](https://cdn-images-1.medium.com/max/800/1*ZvjUtgWUQh7C1jTVaBVjZA.png)
{: .float-left .me-3 }

However, after logging in I wasn’t able to upload a reverse shell for some reason. So I decided to check if any potential exploits were available for it.

![Google Search for exploit](https://cdn-images-1.medium.com/max/800/1*quEzBPg4WgcLKXjiyMAQfA.png)
{: .w-100 }

HMM… Got some interesting results.

After surfing a bit, I got to know that FuelCMS v1.4 has an RCE vulnerability ([CVE-2018–16763](https://www.exploit-db.com/exploits/47138)).

I used [this](https://github.com/ice-wzl/Fuel-1.4.1-RCE-Updated) Python exploit for the Reverse shell.

![Getting an initial foothold](https://cdn-images-1.medium.com/max/800/1*IONVa0Wyd17n70MOlN-g3A.png)
{: .w-100 }

Got the initial foothold with that python exploit. Now we can look around the directories and find the user flag.

![user.txt flag](https://cdn-images-1.medium.com/max/800/1*21Q0R2EZplwNWjmUieaGtQ.jpeg)
{: .float-left .me-3 }

### **Task 2: Root.txt**

I attempted privilege escalation by exploiting SUID and SGID binaries and conducting a system scan with `linpeas`, but I did not uncover any useful information.

So, I decided to take a look at the default web page's files again.

![Web directory files](https://cdn-images-1.medium.com/max/800/1*OHEwgsQJzi4z4YxIN_A4Pg.png)
{: .w-100 }

> Checking the PHP file at `fuel/application/config/database.php`, I got:

![Database Credentials](https://cdn-images-1.medium.com/max/800/1*VgWpUUg2EbQI8h2e53-KSQ.jpeg)
{: .w-100 }

We can try and see if this password is the same for the root user:

![Switching user to root](https://cdn-images-1.medium.com/max/800/1*NRkySWpoIx6Q-fPsN9NRIg.jpeg)
{: .w-100 }

PRIVILEGE ESCALATED! Now we can read the `root.txt` flag.

> cat /root/root.txt

![Machine PWNED](https://cdn-images-1.medium.com/max/800/1*_Cj86Zjqkj6OUD0mctOqZQ.png)
{: .w-100 }

### **Conclusion**

Exploiting this machine was a valuable experience, highlighting that the answer can often be obvious. Whenever we’re stuck, it’s essential to re-examine what we already understand.

Hope y’all liked my first write-up. I tried to keep it concise and insightful. Feedback would be much appreciated.

**See you next time !! ^\_^**

---
*Follow my socials:* [_Github_](https://github.com/Ricktor0), [_Instagram_](https://www.instagram.com/vikr.am121/), [_Linkedin_](https://in.linkedin.com/in/vikram-sharma-957513230)
```