---
title: "THM: OhSINT"
date: 2023-07-16 17:16:10
categories: 
  - TryHackMe
tags:
  - TryHackMe
  - OSINT
---

# Introduction

This is a writeup for the room [OhSINT](https://tryhackme.com/room/ohsint) on TryHackMe. OhSINT is a beginner level room that focuses on OSINT.

This was one of my first rooms that I have successfully completed on TryHackMe. It was a pretty simple room that focused on OSINT.

# Tasks

Image we got:

![Image](./img/ohsint/WindowsXP.jpg)

## 1. What is this users avatar of?

First, I tried to use exiftool to see what properties this image has.

![Image](./img/ohsint/exiftool.png)

This image seems to be attributed to someone named "OWoodflint", and I had it just sent to Google to check them out.

![Image](./img/ohsint/owoodflint.png)

And from the results, I can get their Twitter, GitHub and blog.
So to answer the first question, the user's avatar is of a cat!

## 2. What city is this person in?

I went to dig around their Twitter:

![Image](./img/ohsint/twitter.png)

From here, I can get the BSSID for a wifi point near their house.
Entering it into [Wigle.net](https://wigle.net/), I could get the city they were in.

![Image](./img/ohsint/wigle.png)

## 3. Whats the SSID of the WAP he connected to?

Still on Wigle.net, zooming in into the WAP could give the SSID.

![Image](./img/ohsint/ssid.png)

## 4. What is his personal email address?

I looked around on his blog but didn't manage to get his email unfortunately, so I went on to the third link, which was his GitHub account.

![Image](./img/ohsint/github.png)

Which on his README.md page, contains his email!

## 5. What site did you find his email address on?

Just GitHub!

## 6. Where has he gone on holiday?

I went back to his blog and found that he was in New York, which was where he had gone on his holiday. 

![Image](./img/ohsint/blog.png)

## 7. What is this persons password?
I was a little confused about this question because the hint was checking the source... BUT!
I have a dark reader chrome extension enabled which made me see the password immediately :") when it is supposed to blend into the white background.

![Image](./img/ohsint/blog.png)

# Conclusion

This was quite an interesting room, and I learnt the basic tools to use for OSINT!