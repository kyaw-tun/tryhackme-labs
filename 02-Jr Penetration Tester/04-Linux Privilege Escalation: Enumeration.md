# Linux Privilege Escalation: Enumeration

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Linux Privilege Escalation: Enumeration](https://tryhackme.com/room/linprivenum)

## Introduction

What is Linux Privilege Escalation? It means going from a lower permission user to a higher permission one (mostly root, or sudo user). 

## What is enumeration?

Enumeration is listing the OS properties, user permissions, network configurations, and file permissions to figure out what environment you are working with once you get initial access to a machine. And it is the most important part of the privilege escalation process. 

## OS Enumeration

Command list discussed in the room:

1. `hostname`

I have never used this command personally, because I am always on the TTY based terminal, so, the hostname is always visible. But sometimes, when you are in a TTY less shell, like in metasploit, this command is handy. It reveals hostname of the OS.

It is written simply as this:

```bash
hostname
```

2. `uname`

Now this command, I have used it multiple times, almost every time I update the OS version. There are multiple `uname` commands. `uname -r`, `uname -v` and `uname -a` etc. 

- `-v` signifies kernel version
- `-r` signifies kernel release
- `-a` signifies all

So, after learning that, I just use `uname -a ` all the time to check information about the version or release.

3. `/proc/version`

This is a file, you can read it simply:

```bash
cat /proc/version
```
You can use other commands intead of cat.

This file reveals information about system processes like `uname`. For example, kernel version, build machine that compiled the kernel, compiler used to build the kernel, etc.

4. `/etc/issue`

This file reveals the OS version. And it will come in this way:

```bash
Ubuntu 22.04.1 LTS \n \l
```

5. `ps`

The `ps` command shows the currently running process in the system. 

- `ps aux` - The `aux` option will show processes for all users
- `ps axjf` - The `axjf` option will show processes from all users

6. `cron` service

The `cron` service can be viewed in `etc/crontab` files and the scheduled contents can be viewed on `/var/spool/cron/` directory and `/etc/cron.d/` directory.

7. `dpkg`

The `dpkg` stands for debian package manager.

You can list the installed packages with the command `dpkg -l`. 

## User Enumeration

1. `id`

This command shows the user id, group id, and other relevant identification that the current user belong to in the system. Here are the examples:

```bash
john@home:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),100(users)
john@home:~$ id matt
uid=1002(matt) gid=1002(matt) groups=1002(matt),27(sudo),116(admin)
```

2. `env`

This commmand reveals the environment data/variables about the system.

3. `history`

This command shows the last thousand command history that the user has typed in. And sometimes, you can get some revealing information about the system, but not always.

4. `sudo -l`

This command list the files, and binaries that the user can run as a `sudo` with or without password.  

5. `/etc/passwd`

This file reveals the number of user accounts in the system, both system users and service users.

## Network Enumeration

1. `ifconfig` or `ip a` or `ip addr`

They will show the number of network interfaces of the system.

2. `netstat` or `ss`

This command shows existing communications between services both internal and external. And it has lots of options to gather information on existing connections.

- `netstat -a`
- `netstat -at` or `netstat -au`
- `netstat -l`
- `netstat -s`
- `netstat -tp`
- `netstat -tpln`
- `netstat -i`
- `netstat -ano`

Nowadays, it's been replaced by the `ss` command. And you can check out the functions of the commands in the man page or in help page in terminal, you can get it with `netstat --help` or `ss --help`.

## File Enumeration

1. `ls`

This is in my opinion the most used Linux command along with the `cd`. What it does is list contents inside the directory. Most common ones that go along with the `ls` commands are:

- `ls -l` - Use long listing format
- `ls -la` - Here, `a` means all. So, it will include hidden files and directroies.
- `ls -lah` - Here, `h` means "human readable", so, it will list the size of the content as 4.0K instead of 4096.  

2. `find`

This command list the content you want to search. This command is very useful if you know how to use it. Here are some ways that's mentioned in the room:

- `find . -name flag1.txt`
- `find /home -name flag1.txt`
- `find / -type d -name config`
- `find / -type f -perm 0777`
- `find / -perm -a=x`
- ...

When looking for files with special permissions:

- `find / -perm -u=s -type f 2>/dev/null`
- `find / -perm -g=s -type f 2>/dev/null`
- `find / -perm 4000 -type f 2>/dev/null`
- ...

## Conclusion

When I first did this room, I was already familiar with most of the commands and their usability. But I wasn't familiar with them all. For example, I wasn't familiar with `netstat` or `find`. I knew `find` existed, but I didn't know it could be so much useful during privilege escalation. And now, when revisiting this room after many CTFs, I can say that enumeration is less about knowing a bunch of commands and more about knowing what to look for and why.
