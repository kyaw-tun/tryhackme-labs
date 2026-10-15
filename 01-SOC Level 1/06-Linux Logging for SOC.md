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

## Runtime Monitoring

## Using Auditd

## Conclusion
