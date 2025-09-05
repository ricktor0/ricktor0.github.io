---
title: "How I Hacked NASA"
date: 2024-11-26 10:00:00 +0000
categories: [Bug Bounty, Writeup]
tags: [xss, stored xss, blind xss, nasa, bugcrowd, vdp]
---

Hi everyone!! I’m Vikram, and welcome back to another interesting write-up. Today, I’m excited to share my experience of discovering a stored and blind XSS on one of NASA’s subdomains. This journey has been a mix of learning, perseverance, and, of course, a bit of adrenaline. Let’s dive into it.

PS: I’ve included an introduction sharing my thoughts and motivation behind hunting on this program. If you’re more interested in the technical details and the vulnerability discovery process, feel free to skip ahead to the **“Vulnerability Hunt”** section!

## **Why did I start?**

It all started a few months ago when I kept seeing people proudly sharing NASA appreciation letters on LinkedIn, Instagram, and other platforms. To be honest, seeing them “flex” sparked an urge in me — I wanted one too! I reached out to seniors and friends to ask how to get it and how challenging it might be. Huge thanks to them for motivating me and assuring me that I could do it. While I had reported bugs before, tackling a Vulnerability Disclosure Program (VDP) like NASA’s was an entirely new and exciting experience for me.

**Enough of my story, let's get down to hacking.**

## **The Vulnerability Hunt**

NASA has four main domains in scope for its Vulnerability Disclosure Program (VDP). To begin my assessment, I collected all the subdomains using tools like `Subfinder`, `Sublist3r`, and `Amass`. Once I compiled the list of subdomains, I started manually exploring some of the unique ones to understand their functionality and structure.

While inspecting these subdomains, I came across one of particular interest:
`https://mcl-labcas.jpl.nasa.gov/labcas-ui/s/index.html?search=*`

The website appeared to be tailored for users accessing scientific or research data, with a clean and functional design. During my exploration, I noticed a **custom search functionality** that caught my attention. I was curious to see how it handled input, especially since the search queries were not only reflected on the page but were also stored permanently on the server and reloaded on the interface.

To test for vulnerabilities, I attempted to save an XSS payload by entering `<script>alert("Hello")</script>`. However, the input was sanitized on the client side, preventing the special characters from being saved.

As any pentester or bug hunter would do, I intercepted the request using Caido and manipulated the payload to bypass the client-side filters.

![Intercepting the request with Caido](https://cdn-images-1.medium.com/max/800/1*dxGwFJSO6c596HP5n2eqVA.png)
{: .w-100 }

I tried injecting the following payload:
```html
<script>al\u0065rt('hello')</script>
```
And boom! I found out that it wasn't just reflected in my browser but was also stored on the server. It executed on every browser or device visiting the affected page, confirming it as a **Stored XSS**.

![Stored XSS triggered on page load](https://cdn-images-1.medium.com/max/800/1*jdkGeBnQ4eV6p7A4sdB1VA.png)
{: .w-100 }

### **Buzz Lightyear was right, haha.**

![To infinity and beyond!](https://cdn-images-1.medium.com/max/800/0*BJ4qlheNDwJmQRiq.jpeg)
{: .w-100 }

To demonstrate the impact further, I deployed a **blind XSS** payload via [xss.report](https://xss.report). The payload successfully triggered, capturing sensitive information such as user cookies, IP addresses, and session details, effectively showing how this could be exploited to hijack user accounts.

![Blind XSS payload triggered](https://cdn-images-1.medium.com/max/800/1*69gn9NdAYxpEkAriNtEhgQ.png)
{: .w-100 }
![Sensitive information captured from the blind XSS](https://cdn-images-1.medium.com/max/800/1*8dcykEC0cifSjcJ0YbkAig.png)
{: .w-100 }

## **Chaos**

After gathering all the necessary information, I compiled a detailed report explaining the vulnerability’s impact, along with a clear proof of concept (PoC). But guess what...

![Bugcrowd report marked as P5 Informational](https://cdn-images-1.medium.com/max/800/1*AX4bvqB417H4uBM-DlXyxw.png)
{: .w-100 }

Despite providing a detailed report with a clear PoC, the Bugcrowd triager marked my submission as **P5 (Informational)**, dismissing it as self-XSS. What’s frustrating is that they didn’t even visit the vulnerable page to verify the issue.

> It’s concerning to see a significant vulnerability capable of compromising user data and affecting the organization’s reputation being misclassified. Accurate triaging is essential to ensure impactful vulnerabilities are recognized and properly addressed.

But then, one of my friends who really helped me through the whole process suggested I re-report the vulnerability, as they don't even check the replies to their messages.

**After reporting it again, the vulnerability was finally triaged! It was later fixed, and the affected search-saving functionality was removed entirely.**

![Bugcrowd report triaged successfully](https://cdn-images-1.medium.com/max/800/1*y3G9qyWaRtsb1L5ittkvvw.png)
{: .w-100 }

Getting my first triaged report felt like a dream come true. After a month of reading writeups, watching PoCs on YouTube, and hunting persistently despite moments of doubt, it all paid off. **Receiving that appreciation letter was everything I had worked for.**

![NASA Letter of Appreciation](https://cdn-images-1.medium.com/max/800/1*lPrQJ7CC5sLeS9bbFRCS-g.png)
{: .w-100 }

At the end, to anyone reading this: never lose hope. Keep learning, keep pushing, and trust that your hard work will pay off one day.

Thank you so much for reading my writeup. I hope y’all liked it. I tried my best to share my experience and keep it insightful.

Until next time, Happy Hacking!

---
*Don’t forget to follow my socials:* [_Github_](https://github.com/Ricktor0), [_Instagram_](https://www.instagram.com/vikr.am121/), and [_Linkedin_](https://www.linkedin.com/in/vikram-sharma-957513230?originalSubdomain=in).