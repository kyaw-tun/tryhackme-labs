# Chocolate Factory

**TryHackMe Room:** [Chocolate Factory](https://tryhackme.com/room/chocolatefactory)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

A beginner-friendly TryHackMe room. The goal is to get a foothold on the machine, find the user flag, and escalate to root. I solved it over two days, because my first attempt ran into a few problems, and I have written it up the way it actually happened.

Flags and the specific key and password values are not included.

## Enumeration

My `nmap` scan did not work on the first attempt, so to get moving I guessed that a web server was running and went straight to port `80`. There I found a login form asking for a username and password.

I tried a few basic SQL injection payloads on the login form, but none of them worked. So I moved on to directory enumeration with `gobuster`, which found a hidden directory. Inside it was a page with a command box that let me run commands directly on the system as the `www-data` user.

## Foothold

Using that command box, I looked around the file system and found the key file the room asks about (a Fernet key), and read its contents with `cat`.

From the same command box, I set up a reverse shell and got a proper shell on the machine as `www-data`.

## User access

With a shell, I moved to the `charlie` user’s home directory. The user flag was there, along with several other files, including charlie’s SSH private key and its matching `.pub` file. Apart from the flag, these files were world-readable, which meant I could read charlie’s private key even as `www-data`.

I copied the private key to my own machine, fixed its permissions with `chmod`, and used it to log in as charlie over SSH. That gave me a stable session as the user.

## Privilege escalation

I ran `sudo -l` to see what charlie could run with sudo, and it did not ask for a password. It showed that charlie could run `/usr/bin/vi` as root without a password.

I checked GTFOBins for `vi`, which documents how to break out of it into a root shell when it can be run through sudo. I followed that method and became root.

### The root flag, and a couple of problems

In `/root` there was no `root.txt` like most rooms have. Instead there was a Python script that generates the root flag, but only if you give it the Fernet key from earlier.

This is where I ran into trouble. The script kept failing: it expected the key in a specific byte-string form, and it also depended on the `pyfiglet` module, which was causing an error. I worked out that I could simply remove the `pyfiglet` part of the script, since it was only there for decorative text output, and run it without that. Just as I was about to do this, the machine instance expired, and I had to come back the next day.

On day two, I still had charlie’s private key, so I logged straight back in, edited the script to remove the `pyfiglet` dependency, ran it, and got the root flag.

### The leftover question: charlie’s password

There was one room question I had not answered: charlie’s password. I had not needed it at any point, since the world-readable private key gave me charlie’s account directly, but the room still asks for it.

On day two my `nmap` scan worked, and it showed an FTP port open with anonymous login allowed. I logged in anonymously and downloaded an image file. I ran `steghide` on it and extracted a hidden `b64.txt` file. Decoding the base64 gave me what turned out to be a copy of the `/etc/shadow` file.

The hash for charlie was SHA-512. Cracking that with John the Ripper would have been slow and heavy on my CPU, so I used `hashcat` instead. Even then it took a while, but it did crack.

Here is where it got confusing. The password I cracked did not match the answer the room expected: the room’s answer box showed six characters, and mine was shorter. At first I thought I had made a mistake. But when I logged in as charlie over SSH using the password I had cracked, it worked. So my password was correct for my machine.

Comparing with an older write-up, I saw that their shadow file, and therefore their cracked password, were different from mine. That told me these machine instances are not all identical, and the room’s expected answer came from a different instance than the one I was given. So I was able to crack and verify my own instance’s password, even though it did not match the room’s stored answer.

## Reflection

A fun room, and a good one for practicing a full chain: web foothold, reading a world-readable key, SSH as the user, and a clean sudo privilege escalation through GTFOBins. The parts I learned the most from were the ones that went wrong, working around the broken Python script to still get the flag, and realizing that a cracked password can be correct for your own machine yet not match the room’s expected answer.
