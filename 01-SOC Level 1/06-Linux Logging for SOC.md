# Linux Logging for SOC

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Linux Logging for SOC](https://tryhackme.com/room/linuxloggingforsoc)
> 

## Introduction

Linux has long been a leader in servers and embedded systems, and now its use is even more widespread with the growth of cloud adoption. As a SOC analyst, we are very likely to investigate Linux alerts and incidents, either from traditional on-premises servers or from cloud-native containerized workloads.

This room discuss the most common Linux logs sent to SIEM and learn how to view them directly on-host.

## Working With Text Logs

Linux is not immune to malware and Linux-target intrusions are a growing problem. As a SOC analyst, it is important to investigate Linux alerts, and for this, it is important to understand how its logging works.

- By Linux, the room refers to Linux distributions like Debian, Ubuntu, CentOS, or RHEL
- The room focuses on Linux servers without a GUI and does not explain desktop logging

### Working with Logs

In Linux, you can find system logs in the `var/log/syslog` file. Here is the example:

```bash
cat /var/log/syslog | head
```

You can filter these logs using `grep` command. As in:

```bash
cat /var/log/syslog | grep CRON
```

These logs are often located in `/var/log/` directory. You can find system logs, journal logs, kernel logs, authentication logs, etc. in this directory.

You can search for potential logins across all logs (`/var/log`) with this:

```bash
grep -R -E "auth|login|session" /var/log
```

## Authentication Logs

The useful file for monitoring authentication logs is `/var/log/auth.log` (or `/var/log/secure` in RHEL based systems).

You can filter login and logout events with this:

```bash
cat /var/log/auth.log | grep -E 'session opened|session closed'
```

Sometimes, there might be multiple failed passwords, along with successful one after those. And that might indicate brute-force attack. You can see them in the logs by:

```bash
cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'
```

### Other Miscellaneous Events

There are other events such as adding new users, deleting users, modifying users, or password alterations. You can see this by changing the grep value to `grep -E '(passwd|useradd|usermod|userdel)\['`.  And to see the commands executed, you can filter this by changing the grep value to `grep -E 'COMMAND='`.

## Common Linux Logs

### Generic System Logs

- `/var/log/kern.log` - kernel logs
- `/var/log/syslog (or /var/log/messages)` - system logs (covered above)
- `/var/log/dpkg.log (or /var/log/apt)` - package manager logs for Debian-based systems
- `/var/log/dnf.log (or /var/log/yum.log)` - package manager logs for RHEL-based systems

### App Specific Logs

There are multiple app specific logs: mail logs to investigate phishing, container logs to catch anomalies, database logs to see which queries were run, webserver logs to see which pages were opened, when and by whom. Here is one example the room gave.

```bash
cat /var/log/nginx/access.log
```

### Bash History

There is a file called `.bash_history` and there is a command called `history`. I only knew about the former at first. 

- `.bash_history` gives you the commands from the previous sessions
- `history` command gives you history from previous and current session

## Runtime Monitoring

Runtime events are program launching, file deletion, file creation, modification and network-related events. By default, Linux doesn't log process these events.

Instead, when you need to open a file, create a process, access the camera, or request any other OS service, you make a specific system call. There are over 300 system calls in Linux, like `execve` to execute a program.

All modern EDRs and logging tools rely on them - they monitor the main system calls and log the details in a human-readable format. There is nearly no way for attackers to bypass system calls.

## Using Auditd

Auditd (Audit Daemon) is a built-in auditing solution often used by the SOC team for runtime monitoring. It is not installed on Linux by default. Its instructions for how to use it are located in `/etc/audit/rules.d`.

You can view the generated logs in real time in /var/log/audit/audit.log, but it is easier to use the `ausearch` command, as it formats the output for better readability and supports filtering options. 

Here are some examples:

```bash
ausearch -i -k proc_wget
```

Or when you want to find file events matching the "file_sshconf" key:

```bash
ausearch -i -k file_sshconf
```
- `-i` - Interprets numeric values (UIDs, syscalls, timestamps) into human-readable text
- `-k` - Filters events by the key (tag) defined in the auditd rule
- `proc_wget` - Optional tag specified by engineers in auditd rules that is useful to filter the events
- `file_sshconf` -  Optional tag for a rule that monitors the SSH configuration file, used to filter events related to changes to it

### Auditd Alternatives

Although it provides a verbose logging, it is hard to read and ingest into SIEM. That's why many SOC teams resort to the alternative runtime logging solutions, for example:

- Sysmon for Linux
- Falco
- Osquery
- EDRs

The key to remember is that all listed tools work on the same principle of monitoring system calls. Once you have understood system calls, you will easily learn all the mentioned tools.

## Conclusion
