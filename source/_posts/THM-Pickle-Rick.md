---
title: "THM: Pickle Rick"
date: 2023-12-23 18:33:10
categories: TryHackMe
tags:
  - TryHackMe
  - Linux
  - Privilege Escalation 
---

# Introduction

 This is a writeup for the room [Pickle Rick](https://tryhackme.com/room/picklerick) on [TryHackMe](https://tryhackme.com/). Pickle Rick is a beginner level room which focuses on basic enumeration, Linux filesystem knowledge and privilege escalation. 

I absolutely loved this room, being a Rick and Morty fan myself :D 

# Pre-Enumeration

Firstly, I went to the page itself. The landing page is a pretty simple one:

![Landing Page](./img/picklerick/Landing.png)

As usual, I used inspect element to see if there was anything interesting.

![Inspect Element](./img/picklerick/inspect.png)

Here, we found a comment that contained a username, `R1ckRul3s`. I assume that there is some sort of login page or something similar.

# Reconnaissance

## Nmap

I ran a nmap scan on the machine to see what ports were open with:

```bash
kairos@opensus:~> nmap -sV -sC <ip>
```

The results were:

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-23 18:31 +08
Nmap scan report for <ip>
Host is up (0.33s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4d7e95ccf243d0dfdfc71dad7f6a598d (RSA)
|   256 5901e677a32361e64b748bc86597aa58 (ECDSA)
|_  256 696d23bcda9ab3996ad2d5afc4a0259b (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.08 seconds
```

There are two open ports: 22 and 80. Port 22 is running ssh, and port 80 is running http. I skipped over port 22 since it would only be useful if I had credentials.

## Gobuster

As with the previous rooms, I ran a gobuster scan to see if there were any hidden directories. To make the scan faster, I limited the extensions:

```bash
gobuster dir -u <ip> -w Documents/Tools/wordlists/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,css,js,py
```

The results were:

```
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 315] [--> http://<ip>/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
```

The scan took quite a while (and I had to rerun it a few times due to connectivity issues using dirb :| the order is slightly different from the one above), so I looked at each result as it came in. Looking at `/assets`:

![Assets](./img/picklerick/assets.png)

There wasn't much to see, just a bunch of images, gifs and css/js files. I moved on to the next result.

The next result was `/login.php`.

![Login](./img/picklerick/login.png)

This was the login page that I was looking for! We have the username only though. So let's pin this and circle back to it later.

The next result was `robots.txt`.

Robots.txt is a file that is used on websites to instruct web crawlers what to avoid when crawling to prevent them from indexing certain pages.

The contents usually look something like this (with reference to Google Search Central):

```txt
# Example 1: Block only Googlebot
User-agent: Googlebot
Disallow: /

# Example 2: Block Googlebot and Adsbot
User-agent: Googlebot
User-agent: AdsBot-Google
Disallow: /

# Example 3: Block all crawlers except AdsBot (AdsBot crawlers must be named explicitly)
User-agent: *
Disallow: /
```

But what we see here is:

![Robots.txt](./img/picklerick/robots.png)

So I tried that as the password. And it worked! It led to a page that looked like this:

![Command Panel](./img/picklerick/command.png)

There were other tabs, but all of them were "locked", bringing me to a `denied.php` page.

![Denied](./img/picklerick/denied.png)

Thus, I could only focus on the command panel for now.

# Getting the Flags

Questions:

1. What is the first ingredient that Rick needs?
2. What is the second ingredient in Rick’s potion?
3. What is the last and final ingredient?

## First Ingredient - Command Line

Going to the command panel, I tried running some commands.

```bash
ls
```

Resulted in it showing a bunch of files:

```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

I tried to retrieved the contents of the text files

```bash
cat Sup3rS3cretPickl3Ingred.txt; cat clue.txt
```

But...

![Panel](./img/picklerick/panel.png)

:| Another way then.

I tried to view the file the same way I did with the robots.txt file. Surprisingly, it worked.

`Sup3rS3cretPickl3Ingred.txt` contained the first flag.

```
mr. meeseek hair
```

`clue.txt` contained a hint:

```
Look around the file system for the other ingredient.
```

## Second Ingredient - File System

Hmm... I then tried to use the `tree` command to see the file structure, but it didn't give anything in console. I then tried moving to the home directory first and listing the files there.

```bash
cd /home; ls
```

We got some useful stuff!

```
rick
ubuntu
```

I then moved into the `rick` directory and listed the files there.

```bash
cd rick; ls
```

The output was:

```
second ingredients
```

Since we know that the `cat` command is blacklisted, I tried to use other commands as an alternative. I first tried `head` and `tail`, but they were both blacklisted too. Horrible.

Then I tried the `man` command which can act as a pager to scroll through files.

```bash
man /home/rick/second\ ingredients
```

And we got it!

```
1 jerry tear
```

## Third Ingredient - Root

I looked around the file system for a bit more, but couldn't find much. I then tried to go further up the directory tree.

Doing directory traversal with `../`s and long listing of the files:

```
cd ../../../../ ; ls -la
```

I found

```
total 88
drwxr-xr-x  23 root root  4096 Dec 23 10:29 .
drwxr-xr-x  23 root root  4096 Dec 23 10:29 ..
drwxr-xr-x   2 root root  4096 Nov 14  2018 bin
drwxr-xr-x   3 root root  4096 Nov 14  2018 boot
drwxr-xr-x  14 root root  3260 Dec 23 10:29 dev
drwxr-xr-x  94 root root  4096 Dec 23 10:29 etc
drwxr-xr-x   4 root root  4096 Feb 10  2019 home
lrwxrwxrwx   1 root root    30 Nov 14  2018 initrd.img -> boot/initrd.img-4.4.0-1072-aws
drwxr-xr-x  21 root root  4096 Feb 10  2019 lib
drwxr-xr-x   2 root root  4096 Nov 14  2018 lib64
drwx------   2 root root 16384 Nov 14  2018 lost+found
drwxr-xr-x   2 root root  4096 Nov 14  2018 media
drwxr-xr-x   2 root root  4096 Nov 14  2018 mnt
drwxr-xr-x   2 root root  4096 Nov 14  2018 opt
dr-xr-xr-x 134 root root     0 Dec 23 10:29 proc
drwx------   4 root root  4096 Feb 10  2019 root
drwxr-xr-x  25 root root   880 Dec 23 10:29 run
drwxr-xr-x   2 root root  4096 Nov 14  2018 sbin
drwxr-xr-x   5 root root  4096 Feb 10  2019 snap
drwxr-xr-x   2 root root  4096 Nov 14  2018 srv
dr-xr-xr-x  13 root root     0 Dec 23 10:29 sys
drwxrwxrwt   8 root root  4096 Dec 23 11:39 tmp
drwxr-xr-x  10 root root  4096 Nov 14  2018 usr
drwxr-xr-x  14 root root  4096 Feb 10  2019 var
lrwxrwxrwx   1 root root    27 Nov 14  2018 vmlinuz -> boot/vmlinuz-4.4.0-1072-aws
```

The `root` directory seems interesting. Only superusers can access it.

Using `sudo -l`, I found that I could run `sudo` with no password!

```
Matching Defaults entries for www-data on ip-10-10-177-115.eu-west-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-177-115.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

This means that I would be able to look at the files in the `root` directory.

```bash
sudo  ls /root
```

```
3rd.txt
snap
```

Awesome. Using the same method as before, I found the last flag!

```bash
sudo man /root/3rd.txt
```

```
3rd ingredients: fleeb juice
```

And all the flags have been found!

# Conclusion

All in all, this was a really simple yet fun room. It was great to put my Linux skills to the test and learn on the go. 

I remember learning that the `man` command can be used to page files on accident while studying for my Red Hat Common Test, and it was really cool to see it being used in this room! 