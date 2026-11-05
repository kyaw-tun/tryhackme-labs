# Linux Privilege Escalation (To Do)

This is a list of techniques where you can do privilege escalation to root (or higher privilege user) in the Linux systems. These techniques are mainly from the rooms: [Linux Privilege Escalation](https://tryhackme.com/room/linprivesc) and [Linux PrivEsc](https://tryhackme.com/room/linuxprivesc). But I will keep adding techniques from other rooms as well. The ones worth noting or unusual ones.

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

I have used `LinPeas` and `LinEnum`. The need for the usage of other ones has not come up yet.

## Kernel Exploits

## Sudo

## SUID

## Linux Capabilities

## Cron Jobs

## Linux PATH

## Network File Sharing (NFS)

## ...
