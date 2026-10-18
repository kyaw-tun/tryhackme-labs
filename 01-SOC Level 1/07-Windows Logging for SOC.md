# Windows Logging for SOC

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Windows Logging for SOC](https://tryhackme.com/room/windowsloggingforsoc)
> 

## Introduction

This room introduces how to find logs and interpret Windows event logs. How to use Event viewer, and monitoring log sources like Sysmon and PowerShell.

## What Is Logged

When you start a program, create a file, or just log in to your laptop, the OS processes that event. And then, the OS can log the event by appending a line to some journal, stating the time, action details, and the user behind the action. Every recorded event is called a log, and proper logging ensures that all user and system activity is recorded, thus helping SOC with the following activities:

- Incident Response: Logs can show when and how the attack occurred
- Threat Hunting: Logs allow you to search for signs of malicious activity
- Alerting and Triage: Logs are a building block of any alert or detection rule

### Anatomy of a Log Entry

In Windows, the logs are stored in `C:\Windows\System32\winevt\Logs` directory. Unlike Linux, logs are stored in `.evtx` format and unlike Linux, they are not all intelligible. You need an application like the default-installed `Event Viewer` to read them.

## Security Log: Authentication

Out of all Windows logs enabled by default, the Security event log is the one that brings the most value. And the two most important Security logs are: **Successful Logon (4624)** and **Failed Logon (4625)**.

### Usage of 4624/4625

- Detecting RDP Brute Force

1. Open Security logs and filter for 4625 event ID (Failed login attempts)
2. Look for events with Logon Type 3 and 10 (Network and RDP logins)
   - For most modern systems, the logon type will be 3 (since NLA is enabled by default)
   - For older or misconfigured systems, the logon type will be 10 (since NLA is not used)

3. Every event is now worth your attention, but the main red flags are:
   - Many attempted users like admin, helpdesk,  and cctv (Indicates password spraying)
   - Many login failures on a single account, usually Administrator (Indicates brute force)
   - Workstation Name does not match a corporate pattern (e.g. kali instead of THM-PC-06)
   - Source IP is not expected (e.g. your printer trying to connect to your Windows Server)

- Analyze RDP Logons 

1. Open Security logs and filter for 4624 event ID (Successful logins)
2. Look for events with Logon Type 10 (RDP logins)
   - If NLA is enabled, every RDP logon event is preceded by another 4624 with logon type 3
   - To get a real Workstation Name, you need to check the preceding logon type 3 event
3. Your red flags are either a preceding brute force or a suspicious source IP / hostname
4. If you assume that the login was indeed malicious, find out what happened next:
   - Windows assigns a Logon ID to every successful login (e.g. 0x5D6AC)
   - Logon ID is a unique session identifier. Save it for future analysis!

## Security Log: User Management

## Sysmon: Process Monitoring

## Sysmon: Files and Network

## Powershell: Logging Command

## Conclusion
