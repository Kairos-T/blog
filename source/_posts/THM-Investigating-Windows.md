---
title: "THM: Investigating Windows"
date: 2023-12-25 22:54:39
categories:
  - TryHackMe
tags:
  - Windows
  - Forensics
  - TryHackMe
---

# Introduction

This is a writeup for the room [THM: Investigating Windows](https://tryhackme.com/room/investigatingwindows) on [TryHackMe](https://tryhackme.com/). It relates to Windows forensics. While I am rather interested in digital forensics, I haven't done much of operating system forensics, let alone Windows forensics.

> This is a challenge that is exactly what is says on the tin, there are a few challenges around investigating a windows machine that has been previously compromised.

Connect to the machine using RDP. The credentials the machine are as follows:

Username: Administrator
Password: letmein123!

Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

# Questions

## 1. What is the version and year of the windows machine?

This was rather straightforward. I went into the Windows Settings > System > About and found the version and year of the Windows machine.

![Windows Version](/images/THM-Investigating-Windows/Q1.png)

Ans: Windows Server 2016

Alternatively, I found that you could use a PowerShell command to find the version of the Windows machine.

```powershell
Get-ComputerInfo
```

The answer would be under the `OsName` field.

## 2. Which user logged in last?

Log ons are recorded in the Windows Event Log. In the search bar, I looked for the Event Viewer and opened it.

I then went to `Windows Logs > Security`, which is where the log ons and other security events are recorded. I then looked for the latest log on event.

![Event Viewer](/images/THM-Investigating-Windows/Q2.png)

Ans: Administrator

Additionally, one could filter the logs based on the Event ID. The Event ID for log on events is 4624, and the ID for special privileges assigned to a new log on is 4672.

## 3. When did John log onto the system last? Answer format: MM/DD/YYYY H:MM:SS AM/PM

I filtered the logs for the user `John` and looked for the latest log on event.

![Event Viewer](/images/THM-Investigating-Windows/Q3.png)

Ans: 03/02/2019 5:48:32 PM

Similarly, a better way to do this would be to filter the logs based on the Event ID as mentioned above.

## 4. What IP does the system connect to when it first starts?

The first thing I did was to check out the hosts file. I went to `C:\Windows\System32\drivers\etc` and opened the hosts file in Notepad.

![Hosts File](/images/THM-Investigating-Windows/hf.png)

Looking at the file, there were many entries pointing to the local host. This was a little weird, but we can come back to it in a bit.

I remember seeing something when I started the machine up.

![Startup](/images/THM-Investigating-Windows/startup.png)

It did show what IP it w as connecting to!

Ans: 10.34.2.3

Frankly, this was quite easy to miss since it was a popup that disappeared quickly. Another way would be to look at the `regedit`. In regedit, and at `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`. This is where startup programs are stored.

![Regedit](/images/THM-Investigating-Windows/Q4.png)

## 5. What two accounts had administrative privileges (other than the Administrator user)? Answer format: username1, username2

Initially, when answering the first question, I snooped around the users settings too. I saw two other accounts, John and Jenny.

![Users](/images/THM-Investigating-Windows/Settings.png)

Sadly, they were not the answers. I searched up commands to find the answer.

```cmd
net localgroup administrators
```

![Admins](/images/THM-Investigating-Windows/Q5.png)

Ans: Jenny, Guest

## 6. Whats the name of the scheduled task that is malicious?

For this task, I looked around in the Task Scheduler > Task Scheduler Library. There were a few tasks:

![Tasks](/images/THM-Investigating-Windows/Q6.png)

Here, the one that stood out and could fit the answer blank was the `Clean file system`.

Ans: Clean file system

## 7. What file was the task trying to run daily?

Looking at the `Actions` tab of the task, it was trying to run a file `nc.ps1`.

Ans: nc.ps1

## 8. What port did this file listen locally for?

Also in the `Actions` tab, we can see that the file was listening on port 1348.

## 9. When did Jenny last logon?

In a similar fashion to Q3, I filtered the logs for the user `Jenny` and looked for the latest log on event. Except... there were no log on events for Jenny.

Ans: Never

## 10. At what date did the compromise take place? Answer format: MM/DD/YYYY

Throughout the time I was on the machine, there was a program that kept popping  up.

![Program](/images/THM-Investigating-Windows/Popup.png)

It showed the path of the program, `C:\TMP\mim.exe`. So, the date of the compromise would probably be when the file/folder was created. Navigating to the folder:

![Folder](/images/THM-Investigating-Windows/Q10.png)

Ans: 03/02/2019


## 11. During the compromise, at what time did Windows first assign special privileges to a new logon? MM/DD/YYYY HH:MM:SS AM/PM

We know that the compromise took place on 03/02/2019, and around 4.37pm. So, I filtered the logs for event ID 4672 and between around 4.00pm to 5pm.

After I filtered it, and found the latest event, I got a wrong answer :(

I tried filtering only for Security events the same way and I got the answer:

![Event](/images/THM-Investigating-Windows/Q11.png)

After looking back at the question, I realised that the issue was the Task Category rather than the type of event. It should be `Security Group Management` instead of `Special Logon`. Security Group Management (Special Logon) is apparently different from Special Logon! 

Special logon: User logon sessions with special privileges
Security Group Management (Special Logon): User with special privileges logs on AND performs an action that changes security groups/group memberships. 

So, we should be searching for Security Group Management (Special Logon) events instead since we are looking for the time the logon was **assigned** special privileges.

Ans: 03/02/2019 4:04:49 PM

## 12. What tool was used to get Windows passwords?

Earlier on, in the task scheduler, I found other tasks. I looked through the `Actions` tab of each of the tasks and one of them, `GameOver` had one that seemed to take the passwords. Anyway, I've been seeing the same pop up every 5 minutes, but it didn't last long enough for me to read it.

![GameOver](/images/THM-Investigating-Windows/Q12a.png)

Since the task was running `mim.exe` in the same folder earlier, I checked out the files there. I tried running the file again but the pop up disappeared too quickly. Anyway, I found a file `mim-out` which seemed to be the output of the program.

![mim-out](/images/THM-Investigating-Windows/Q12b.png)

Ans: mimikatz

## 13. What was the attackers external control and command servers IP?

While doing task 4, the host file looked weird.

![Hosts File](/images/THM-Investigating-Windows/Q13.png)

In Command Prompt, I tried pinging google.com.
```powershell
C:\Users\Administrator>ping google.com

Pinging google.com [76.32.97.132] with 32 bytes of data:
Request timed out.

Ping statistics for 76.32.97.132:
    Packets: Sent = 1, Received = 0, Lost = 1 (100% loss),
Control-C
```
It was unreachable! On my host device, when I tried pinging google.com:

```bash
kairos@opensus:~> ping google.com
PING google.com (64.233.170.101) 56(84) bytes of data.
64 bytes from sg-in-f101.1e100.net (64.233.170.101): icmp_seq=1 ttl=102 time=7.09 ms
```

It was a different IP :O Seemed like the C2 server here!

Ans: 76.32.97.132

## 14. What was the extension name of the shell uploaded via the servers website?

On Windows, the default webserver is IIS (Internet Information Services). The default directory for it is `C:\inetpub\wwwroot`. I navigated to the directory.  

![IIS](/images/THM-Investigating-Windows/Q14.png)

There were two types of files, `.JSP` and `.GIF`. Obviously, the shell was not a `.GIF` file. I had to search up what `.JSP` was. 

> JavaServer Pages (JSP) is a technology that helps software developers create dynamically generated web pages. JSP is similar to PHP and ASP, but it uses the Java programming language.

Knowing that PHP reverse shells exist, I searched up more about JSP shells and it seems like they exist too, and is a common web attack vector.

Ans: .jsp

## 15. What was the last port the attacker opened?

Knowing little about Windows, I had to use the hint. 
> Firewall

I went to Windows Defender Firewall > Monitoring > Firewall to look for the latest rule. Looking at the `Local Port` column, I found the latest rule:

![Firewall](/images/THM-Investigating-Windows/Q15.png)

Ans: 1337

## 16. Check for DNS poisoning, what site was targeted?

DNS poisoning, or DNS spoofing, is a way to  redirect traffic  to an attacker's website.

> One way to perform a DNS poison is to modify the host file that’s located on each individual device. The host file in the machine takes precedence over any DNS queries, so it doesn’t matter what is configured in a DNS. Your computer is going to follow whatever is listed in that host file.

That seems like what was happening in the host file earlier.

Ans: google.com

# Conclusion

All in all, this was a great learning experience for me, especially as someone who has very little experience with Windows forensics and cmd/Powershell. 

I probably should brush up on my Windows knowledge!