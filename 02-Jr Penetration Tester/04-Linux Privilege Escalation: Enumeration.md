# Linux Privilege Escalation: Enumeration

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Linux Privilege Escalation: Enumeration](https://tryhackme.com/room/linprivenum)

## Introduction

What is Linux Privilege Escalation? It means going from a lower-privileged user to a higher-privileged one, usually `root`. 

## What is enumeration?

Enumeration is the process of gathering information about the OS, user permissions, network configuration, file permissions, and other aspects of the system to figure out what environment you are working with once you get initial access to a machine. And it is one of the most important parts of the privilege escalation process.

## OS Enumeration

Command list discussed in the room:

1. `hostname`

I have never used this command personally, because I am usually on a TTY-based terminal where the hostname is already visible in the prompt. But sometimes, when you are in a TTY-less shell, this command is handy because the hostname may not be visible. And this command reveals hostname of the system.

It is written simply as this:

```bash
hostname
```

2. `uname`

Now this command, I have used it multiple times, almost every time I want to check my kernel information. There are multiple `uname` commands. `uname -r`, `uname -v` and `uname -a` etc. 

- `-r` signifies kernel release
- `-v` signifies kernel version
- `-a` signifies all

So, after learning that, I just use `uname -a` all the time to quickly check information about the system and kernel.

3. `/proc/version`

This is a file, you can read it simply:

```bash
cat /proc/version
```
You can use other commands instead of `cat`.

This file reveals information about the Linux kernel and how it was built. For example, it shows the kernel version, the user or machine that compiled it, and the compiler used to build the kernel.

4. `/etc/issue`

This file reveals the OS version. And it will come in this way:

```bash
Ubuntu 22.04.1 LTS \n \l
```

5. `ps`

The `ps` command displays information about running processes.

- `ps aux` - Shows running processes from all users in a user-oriented format.
- `ps axjf` - Shows processes from all users, including processes without a terminal, in a hierarchical format.

6. `cron` service

The `cron` jobs can be found in `/etc/crontab` files and the scheduled contents can be viewed on `/var/spool/cron/` directory and `/etc/cron.d/` directory.

7. `dpkg`

`dpkg` is the low-level package management tool used by Debian-based Linux distributions. It is used to install, remove, inspect, and manage `.deb` packages. A `.deb` file is the actual package file, somewhat similar to an `.msi` installer on Windows.

You can list the installed packages with the command `dpkg -l`. 

## User Enumeration

1. `id`

This command shows the user ID, group ID, and other relevant information about the current user and the groups they belong to. Here are the examples:

```bash
john@home:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),100(users)
john@home:~$ id matt
uid=1002(matt) gid=1002(matt) groups=1002(matt),27(sudo),116(admin)
```

2. `env`

This command reveals the environment data/variables about the system.

3. `history`

This command shows commands that the user has previously entered. Sometimes, you can find useful or sensitive information in the history, but not always.

4. `sudo -l`

This command lists the commands that the current user is allowed to run with `sudo`, including whether a password is required.

5. `/etc/passwd`

This file contains information about user accounts on the system, including regular users, system users, and service accounts.

## Network Enumeration

1. `ifconfig` or `ip a` or `ip addr`

These commands display information about the system's network interfaces, including their IP addresses and other configuration details.

2. `netstat` or `ss`

These commands can display information about network connections, listening ports, sockets, and other network-related information.

```text
netstat -a    # Show all sockets
netstat -at   # Show TCP sockets
netstat -au   # Show UDP sockets
netstat -l    # Show listening sockets
netstat -s    # Show network statistics
netstat -tp   # Show TCP connections with process information
netstat -tpln # Show listening TCP sockets numerically with process information
```

Nowadays, netstat is considered deprecated on many Linux distributions, and ss is generally preferred instead. And you can check out the functions of the commands in the man page or in help page in terminal, you can get it with `netstat --help` or `ss --help`.

## File Enumeration

1. `ls`

This is, in my opinion, the most used Linux command along with the `cd`. It lists contents inside the directory. Most common ones that go along with the `ls` commands are:

- `ls -l` - Use long listing format
- `ls -la` - Here, `a` means all. So, it will include hidden files and directories.
- `ls -lah` - Here, `h` means "human readable", so, it will list the size of the content as 4.0K instead of 4096.  

2. `find`

This command searches for files and directories based on different criteria. This command is very useful if you know how to use it. Here are some ways that's mentioned in the room:

- `find . -name flag1.txt`
- `find /home -name flag1.txt`
- `find / -type d -name config`
- `find / -type f -perm 0777`
- `find / -perm -a=x`
- ...

When looking for files with special permissions, such as SUID and SGID:

- `find / -perm -u=s -type f 2>/dev/null`
- `find / -perm -g=s -type f 2>/dev/null`
- `find / -perm 4000 -type f 2>/dev/null`
- ...

## Conclusion

When I first did this room, I was already familiar with most of the commands and their usability. But I wasn't familiar with them all. For example, I wasn't familiar with `netstat` or `find`. I knew `find` existed, but I didn't know it could be so useful during privilege escalation. And now, when revisiting this room after many CTFs, I can say that enumeration is less about knowing a bunch of commands and more about knowing what to look for and why.
