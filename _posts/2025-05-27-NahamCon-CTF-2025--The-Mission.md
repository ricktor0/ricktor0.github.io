---
title: "NahamCon CTF 2025: “The Mission”"
date: 2025-05-27 10:00:00 +0000
categories: [CTF, Writeup]
tags: [nahamcon, web challenge, graphql, heapdump, 403-bypass, spring-boot, ctf-writeup]
---

A deep dive into one of the most thrilling web challenges from NahamCon CTF 2025—GraphQL injections, 403 bypasses, heapdump leaks, and more.

## **Introduction**

Hi everyone! I’m Vikram and I’m back with something exciting. This time, my team **0bscuri7y** and I participated in **NahamCon CTF 2025**, and we proudly secured the **13th position** among some of the best players out there!

Huge shoutout to my awesome teammates—**Bhavya, Shubham, Rishabh, Shivendoo, Vatsal, and Vaibhav** for pulling this off.

![Team 0bscuri7y 13th place at NahamCon CTF 2025](https://cdn-images-1.medium.com/max/800/1*hwOyGTR4Gs-K9OoO_qh-Zw.png)
{: .w-100 }

**NahamCon CTF 2025** delivered an incredible web challenge titled “The Mission”, blending classic web exploitation techniques with a compelling narrative.

In this write-up, I walk through my step-by-step approach to solving the challenge. With six cleverly hidden flags, realistic attack surfaces, and plenty of “aha!” moments, this challenge was a masterclass in modern bug bounty tactics. Here’s how I tackled it, one exploit at a time.

## **Challenge Description**

> Welcome to the BugBounty platform, you’ve submitted a report for Yahoo but oh no!! So has STÖK and it looks like you’re going to get dupped!
> Hack into the platform and change STÖK’s report to duplicate first so you can grab the bounty!
> You can login with the following details:
> _Username: hacker_
> _Password: password123_

In this challenge, I take on the role of a bug bounty hunter competing for a payout on a simulated platform. The catch? Another well-known researcher, STÖK, has submitted the same vulnerability report. To make sure I walk away with the bounty, I need to dig into the platform, find a way to tamper with the system, and flag STÖK’s report as a duplicate before mine is.

There were 6 Flags in total to complete this challenge. The challenge also provided a `wordlist.txt` file.

---

## **The Mission Begins**

## **Flag 1: Spoke too soon :)**

After spinning up an instance, you’re given a website which looks like this:

![The Mission CTF Homepage](https://cdn-images-1.medium.com/max/800/0*B6q-nDPVfrpUpyNx)
{: .w-100 }

The website had sections like Home, Top Hackers, and Login. But just like any CTF, I checked `/robots.txt` first and guess what?

![robots.txt revealing the first flag](https://cdn-images-1.medium.com/max/800/0*vEl64yfYIn3LiaW1)
{: .w-100 }

Bingo! I got **Flag #1** and an interesting looking endpoint. I was like "Eh! that easy?" But maybe I spoke too soon.

## **Flag 2: When in Doubt, Wordlist It Out**

I couldn’t just walk past that `/internal-dash` login page without poking it. I visited `/internal-dash/` which redirected me to a login page titled "Internal Dashboard". My inner hacker instincts screamed “Admin Panel!”

![Internal Dashboard Login Page](https://cdn-images-1.medium.com/max/800/0*thzleca-dezYvY--)
{: .w-100 }

I threw everything I had at it: Default creds? Nope. SQLi? Nah. Session tampering? I wish! Remembering the `wordlist.txt` file, I loaded it into DirBuster and started brute-forcing directories.

![DirBuster directory brute-forcing](https://cdn-images-1.medium.com/max/800/0*DQReBfCsGMXpVopE)
{: .w-100 }

Soon enough, I found `/api/v1`—and it threw a stack trace revealing something juicy: **openjdk:19-jdk.**

![Stack trace revealing openjdk version](https://cdn-images-1.medium.com/max/800/0*1XNh9FsFN1HJhyuN)
{: .w-100 }

After some research, I learned about the `/actuator` endpoint in Spring Boot applications, which can expose sensitive internal data. But when I tried to access it, I got a 403 Forbidden.

![403 Forbidden on /actuator endpoint](https://cdn-images-1.medium.com/max/800/0*3w8RrhwH0uPB5gNJ)
{: .w-100 }

A WAF? Maybe I was on the right track. I quickly used a 403 bypass plugin in Caido.

![Bypassing 403 with Caido](https://cdn-images-1.medium.com/max/800/0*urN8G7uG8pi96U4I)
{: .w-100 }

Bypassed that 403 with a sneaky character encoding, and yes sir! I was in. I grabbed **Flag #2** and stumbled upon another juicy file: `/heapdump`.

![Accessing /actuator and finding the heapdump](https://cdn-images-1.medium.com/max/800/0*6Eu0LMCLvO5SVg9U)
{: .w-100 }

## **Flag 3: Made It In... kinda**

I downloaded the heapdump and it was like finding a treasure map.

![Heapdump download](https://cdn-images-1.medium.com/max/800/0*CV_XHy0F7fmGwaSY)
{: .w-100 }

Looking closely at the heapdump, I realized it contained a POST request. I replayed the request and boom, got a **valid internal dashboard token** in return.

![Replaying request to get internal token](https://cdn-images-1.medium.com/max/800/0*UgXhRRQC2-Y9phkT)
{: .w-100 }

After checking the `/logout` endpoint, I noticed a cookie named `int-token` was being cleared. So, I took the token from the heapdump and set it as the `int-token` cookie in a request to `/internal-dash/`.
```http
GET /internal-dash/ HTTP/1.1
Host: challenge.nahamcon.com:30510
Cookie: int-token=a1c2860d05f004f9ac6b0626277b1c36e0d30d66bb168f0a56a53ce12f3f0f7a
...
```
This time, it worked! I was logged into the internal "Bug Bounty Report Lookup" dashboard.

![Successfully logged into the internal dashboard](https://cdn-images-1.medium.com/max/800/0*HINkmsnckAGMbrqk)
{: .w-100 }

That got me **Flag #3**. However, the dashboard required a `report_id`, which I didn’t have yet.

## **Flag 4: Peeking Where I Shouldn’t (Thanks, GraphQL)**

Back on the main domain, I used the provided credentials (`hacker`:`password123`) to log in.

![Main user dashboard](https://cdn-images-1.medium.com/max/800/0*yRyhKb6Qv2wQK3F9)
{: .w-100 }

Inspecting the network traffic, I found a `/api/v2/graphql` endpoint. It looked like it might be vulnerable to **GraphQL injection**, and spoiler alert: IT WAS.

![GraphQL endpoint found in network traffic](https://cdn-images-1.medium.com/max/800/0*YOyXGR375DbMQT0P)
{: .w-100 }

I sent this classic test payload:
```graphql
{"query": "{ users { id username email } }"}
```
To my surprise, the server responded without any authentication checks, returning a full list of registered users.

![GraphQL injection leaking user data](https://cdn-images-1.medium.com/max/800/0*aDail-GW_PLzSZAS)
{: .w-100 }

Got **Flag #4**! Out of all the users, one stood out: **Stok**. I grabbed his user ID and used the `/api/v2/reports` endpoint to get all his submitted reports.

![Finding Stok's report via API](https://cdn-images-1.medium.com/max/800/0*OgI-SPVnHa1mAcaP)
{: .w-100 }

Now I had two crucial things:
1.  **Stok’s ID:** `15ee453d-18c7–419b-a3a6-ef8f2cc1271f`
2.  **Stok’s Report ID:** `c03dd42e-d929–4a50–9a8e-1ab6b2dd5e8a`

Let’s jump back to the internal dashboard.

## **Flag 5: Wait, there’s more??**

Back on the internal dashboard, I entered Stok’s report ID but was hit with a 403 Forbidden. Even with a valid token, I wasn’t authorized to view that report. Digging into the dashboard’s source code, I noticed a `change_hash` parameter was required to update a report's status, which I couldn't get due to the 403.

After some experimentation with the `id` parameter, passing `id` as `x/../../../search?q=<REPORT_ID>` tricked the server into leaking the `change_hash`.

![Leaking the change_hash via path traversal](https://cdn-images-1.medium.com/max/800/0*R5dDa6_Mu2TPNdrj)
{: .w-100 }

With the hash, I was able to modify the report’s status. I quickly marked Stok’s report as "Duplicate"...

![Marking Stok's report as Duplicate](https://cdn-images-1.medium.com/max/800/0*mUM1mLjEbETdbUO7)
{: .w-100 }

...and updated mine to "Accepted".

![Marking my report as Accepted](https://cdn-images-1.medium.com/max/800/0*rb0J9sv8bn1bfsKT)
{: .-100 }

After logging back into the main dashboard, there it was: **Flag #5**. A clean finish to a sneaky privilege escalation chain.

![Fifth flag captured](https://cdn-images-1.medium.com/max/800/0*0q8kR8l7BBj5YR22)
{: .w-100 }

## **Flag 6: Am I Adam Langley?**

Just when I thought I was done, there was one last **bonus flag**. I noticed a "Bug Bounty Assistant" bot on the dashboard. What if I pretended to be someone else? I spoofed my name to **Adam Langley** and tricked the bot.

![Tricking the bot by spoofing name](https://cdn-images-1.medium.com/max/800/0*2WkhT_GxxsjixbJ3)
{: .w-100 }

**Boom!** It spilled the final secret, and **Flag #6** was secured.

## **Closing Thoughts & Shoutouts**

Big thanks to **Adam Langley** for crafting such a fun, challenging, and beautifully designed CTF. And a shout-out to **John Hammond** and **Ben Sadeghipour** for organizing **NahamCon 2025** and pulling it all off so smoothly.

**Happy hacking and see you next time with more cool stuff! 🚀**

---
**Don’t forget to connect on** [**Linkedin**](https://in.linkedin.com/in/vikram-sharma-957513230) **and** [**X**](https://x.com/vikramhere200).