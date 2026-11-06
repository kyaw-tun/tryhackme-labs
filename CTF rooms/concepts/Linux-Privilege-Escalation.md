# Linux Privilege Escalation (Ongoing)

This is a list of techniques for privilege escalation to root (or a higher-privileged user) on Linux systems.. These techniques are mainly from the rooms: [Linux Privilege Escalation](https://tryhackme.com/room/linprivesc) and [Linux PrivEsc](https://tryhackme.com/room/linuxprivesc). But I will keep adding techniques from other rooms as well. The ones worth noting or unusual ones.

## Introduction

What is Linux Privilege Escalation? It is when a user moves from a lower-permission account to a higher-permission one, usually root, by exploiting a vulnerability, misconfiguration, or flaw to gain access to files or resources normally restricted to them.

## Enumeration

Enumeration is important in the privilege escalation. Because it tells us what we are dealing with once we get initial access. Here are the basic commands in Linux for enumerating the system, and their description:

| Commands | Description |
| -------- | ----------- |
| `hostname` | Shows the hostname of the system |
| `uname -a` | Shows the kernel version, build number of the system |
| `/proc/version` | Shows the kernel version and compiler details (read from a file) |
| `/etc/issue` | Shows the system’s identification / pre-login banner text (often the OS and version) |
| `ps` | Shows information about processes and their status | 
| `env` | Shows the environment variables, including `PATH` |
| `sudo -l` | Shows information about the list of things the user is permitted to run `sudo` on |
| `ls` | Shows information about files in the current working directory |
| `id` | Shows general overview of the user privilege level and group membership |
| `/etc/passwd` | Shows user accounts in the system (including services accounts and root account) |
| `history` | Shows earlier commands the user has typed | 
| `ifconfig` | Shows information about network interfaces in the system |
| `netstat` | Shows information on existing connections |
| `find` | Shows important information and potential privilege escalation vectors (but you have to know how to use one) |

These are the basic commands you should know, and know how to use them. But if you think, they are time consuming, there are automated ways to enumerate the system. The popular ones are:

- [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester)
- [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- [Linux Priv Checker](https://github.com/linted/linuxprivchecker)

I have used `LinPeas` and `LinEnum`. The need for the usage of other ones has not come up yet.For a fuller walkthrough of enumeration, see my [enumeration write-up](../../02-Jr%20Penetration%20Tester/04-Linux%20Privilege%20Escalation%3A%20Enumeration.md).

## Kernel Exploits

This method should be the last thing on a pentester's mind when escalating privileges. Because, even if it does work or it doesn't, it can crash or destabilize the system. 

Kernel exploits mean exactly what it's written, exploiting the Linux kernel. More often or not, Linux server machines run outdated Linux systems (at least in CTFs). And those outdated kernels would have vulnerabilities and exploits ready that can be found online.

For this task, the lab machine was running `Ubuntu 14.04 LTS` with Kernel version `3.13.0-24-generic`, and in the previous task (which i didn't explain), we were asked to find a `CVE` vulnerability code name for that specific kernel, and now, we are asked to gain root, using the exploit found online.

It can be pretty easy, if you know what you are looking. First place to look for is almost always "exploit-db" site. For me, I first looked at the wrong one there, and I saw "Metasploit required", so, i checked Metasploit and looked for the listing there. It might have been definitely possible, but number of payloads I saw there overwhelmed me for a bit, and I tried to look for another one on "exploit-db". And I found one `.c` code there. 

How it works is, first you download/copy the `.c` code, and save it, and compile it using `gcc` and then run the binary using `./ofs`. And that's how I gained root.  

## Sudo

This is almost always the first thing you check when you get initial access to a system. Sometimes, you would see a road block like asking password for the system when you don't know one. But sometimes, you get the initial access through a weak password, and when running `sudo -l`, you might find that the user has `sudo` privileges if not all.

For this task, the teaching and the task questions don't match. The teaching explains about how to get root using `LD_PRELOAD` (which I'll get to in the future), but the questions are about `sudo` permissions that the machine user is allowed to run and about how to get to root using `nmap`. When I first do this room, I might have been really confused (and no wonder, it took me two to three days back then). Now, I know what `LD_PRELOAD` is (although, my knowledge still needs to be put on test), and I know how to get to root using `nmap` (for some versions only) and the easiest part, the programs the user of the machine can run `sudo` on.

And one of the questions of this task also ask the hash of one user's password. It is easy enough too.

## SUID

`SUID` (Set User ID) and `SGID` (Set Group ID) are special Linux/Unix file permissions that allow an executable program to run with the privileges of its owner or group rather than the user running it.

I only found out about `SUID` when doing Red Team Capstone or when learning about Red Team Essentials. Now, I have used it to get to root in multiple occasions. And I still have yet to encounter any privilege escalation using `SGID`. You just cannot `sudo -l` the SUID or SGID like you can with `sudo`. You have to enumerate it manually or with automated tools. Here is one useful way using `find`:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

And when you find unusual binary that are SUID set, you can check out ways to get to root using GTFObins, it might be file read, or write, or even getting shell, it depends on the binary. 

## Linux Capabilities

## Cron Jobs

## Linux PATH

## Network File Sharing (NFS)

## ...
> *This write-up is in progress; more techniques are being added.
