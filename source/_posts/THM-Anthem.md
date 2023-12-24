---
title: "THM: Anthem"
date: 2023-12-24 15:06:14
categories:
  - TryHackMe
tags:
  - TryHackMe
  - Windows
---

# Introduction

 This is a writeup for the room [Anthem](https://tryhackme.com/room/anthem) on TryHackMe. Anthem is a beginner level room that focuses on Windows exploitation.

This was my first ever Windows room that I have successfully completed. It was pretty simple and straightforward!

# Enumeration

Questions:

1. Let's run nmap and check what ports are open.
2. What port is for the web server?
3. What port is for remote desktop service?
4. What is a possible password in one of the pages web crawlers check for?
5. What CMS is the website using?
6. What is the domain of the website?
7. What's the name of the Administrator
8. Can we find find the email address of the administrator?

## Nmap - Q1, 2, 3

Starting off with a simple nmap scan:

```bash
nmap -sC -sV -oN nmap  <ip>
```

The results were:

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-24 15:05 +08
Nmap scan report for <ip>
Host is up (0.34s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Anthem.com - Welcome to our blog
| http-robots.txt: 4 disallowed entries
|_/bin/ /config/ /umbraco/ /umbraco_client/
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-12-24T07:05:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2023-12-23T06:59:49
|_Not valid after:  2024-06-23T06:59:49
| rdp-ntlm-info:
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2023-12-24T07:05:52+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.38 seconds
```

Here, we can see that there are two open ports, 80 and 3389. Port 80 is running HTTP, and there was a robots.txt file. As for 3389, a quick Google search showed that port 3389 is:

> Remote Desktop Protocol (RDP) is a Microsoft proprietary protocol that enables remote connections to other computers, typically over TCP port 3389.

So that gave us the answers to questions 2 and 3.

# Website Analysis

On first glance, the website looks pretty normal. It seems to be a page for a blog:
![landing page](./img/anthem/landing.png)

I clicked around the different pages.

**Article**
Looking at the first article:
![article](./img/anthem/article.png)
We can see an author and their email, `JD@anthem.com`. I pinned that first, since it may come in handy later.

**Categories and Tags**
Looking at the categories tab:
![categories](./img/anthem/categories.png)

It was just empty, and so was the tags tab.
![tags](./img/anthem/tags.png)
So, I moved on.

**Robots.txt - Q4, 5, 6**
In the nmap scan, we saw that there was a robots.txt file. I checked it out by going to `http://<ip>/robots.txt`:
![robots.txt](./img/anthem/robots.png)

The `robots.txt` file did look normal... except for the first entry. I thought that it could be the possible password in question 4, and it was indeed it :)

Moving on, there were other entries, `/bin/`, `/config/`, `/umbraco/`, and `/umbraco_client/`.

I tried going to `/bin/` and `/config/`, but `/bin` couldn't be accessed, and `/config/` just redirected me to the main page.

Next were the Umbraco pages. I've never heard of it before, but a quick search showed that it was a CMS (Content Management System). That answered question 5 (What CMS is the website using?).

**Administator - Q7, 8**

There was an article that mentioned the admin briefly, so I went back to check it out.

```
A cheers to our IT department
TUESDAY, DECEMBER 31, 2019
During our hard times our beloved admin managed to save our business by redesigning the entire website.

As we all around here knows how much I love writing poems I decided to write one about him:

Born on a Monday,
Christened on Tuesday,
Married on Wednesday,
Took ill on Thursday,
Grew worse on Friday,
Died on Saturday,
Buried on Sunday.
That was the end…
```

Writing a poem about the admin... Simply searching up the poem gave the answer to question 7.

![poem](./img/anthem/poem.png)

As for the email, remember the email we found earlier? Using the same format, we can find the email of the admin, `SG@anthem.com`.

# Spot the Flags

Our beloved admin left some flags behind that we require to gather before we proceed to the next task..

1. What is flag 1?
2. What is flag 2?
3. What is flag 3?
4. What is flag 4?

At this point, I realised that I had completely forgotten to use inspect element. But this time, I just used curl to get the source code.

I started from where I was, and found a flag:

![flag4](./img/anthem/flag4.png)

Funnily enough, I found the last flag first. Anyway, I went back to the other pages and found the flags:

![flag1](./img/anthem/flag1.png)
Which gave us two flags, `HM{L0L_WH0_US3S_M3T4}` and `THM{G!T_G00D}`.

There was one more flag remaining, and I looked through the different pages and realised I had missed out the profile page of the author.

![jd](./img/anthem/jd.png)
![flag3](./img/anthem/flag3.png)

# Final Stage

Questions:

1. Let's figure out the username and password to log in to the box.(The box is not on a domain)
2. Gain initial access to the machine, what is the contents of user.txt?
3. Can we spot the admin password?
4. Escalate your privileges to root, what is the contents of root.txt?

Here is where it gets more interesting.

## Login - Q1

Since we found that the port 3389 was open, we can try to connect to it using `rdesktop`:

```bash
rdesktop <ip>
```

This prompts for a username and password. We do know the password, but not the username exactly. I tried `sg` with the password found in the `robots.txt` file, and it logged us in.

## User Flag - Q2

![rdesktop](./img/anthem/win.png)

On the desktop, there was a file `user.txt`. Clicking it gave us a flag.

![user](./img/anthem/user.png)

## Admin Password - Q3

I had to rack my brains and eventually took the hint.

> It is hidden.
> Fair enough. I do know that on Linux, there are hidden files and folders that start with a `.`. There is also usually a feature that allows you to view hidden files and folders.

I searched up how to view hidden files and folders on Windows, and found that you can do it in File Explorer too! View > Show/Hide > Hidden Items.

![hidden](./img/anthem/hiddenfiles.png)

One folder was hidden, `C:\backup`. Inside, there was a file `restore.txt`. Double clicking it showed us that... we lacked permissions to view it.

![restore](./img/anthem/restore_error.png)

I looked at the `Properties` of the file, and realised that I could simply change the permissions to allow me to view the file.

![permissions](./img/anthem/restore_perm.png)
![permissions](./img/anthem/restore_perms.png)

Essentially, I added `sg` to the list of users that could view the file.

Opening the file, we can see that there is a password.

![restore](./img/anthem/restore.png)

This gave us the answer to question 3 of the final stage.

## Root Flag - Q4

So now we have the admin password, and we have to login as the admin.

I went to the `C:\Users\Administrator` folder in File Explorer, and double clicked on it, which prompted me for the password.

![admin login](./img/anthem/admin_login.png)

Now, we can access to the Admin's files.
![admin](./img/anthem/user⁄admin.png)

I used the search bar to search for `root.txt`, and found it in `C:\Users\Administrator\Desktop`.

![root](./img/anthem/root.png)
![root](./img/anthem/final_flag.png)

And that was our final flag!!

# Conclusion

I do think this was a pretty simple room, and it was a good introduction to Windows exploitation. It taught me how to use `rdesktop`, and basic Windows
stuff like hidden files and folders, changing permissions, and admin privileges.