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

| Event ID | Description |
| -------- | ----------- |
| 4720 / 4722 / 4738 | A user account was created/enabled/changed |
| 4725 / 4726 | A user account was disabled/deleted |
| 4723 / 4724 | A user changed their password / User's password was reset |
| 4732 / 4733 | A user was added to / removed from a security group |

These event IDs above are normal in user management IDs. However, these events can be triggered during malicious usage too such as: attacker creating a backdoor account or enabling an old one to avoid detection (4720/4722/4738), disabling privileged SOC accounts to slow down their actions (4725/4726), etc.

All user management events have similar structure, and can be split up in three: 
- Subject: who did the action
- Object: who was the target
- Details: what changes were made

### Hunt for Backdoored Users

1. Open Security logs and filter for 4720 / 4732 event IDs

2. Manually review every event; the red flags are:
   - No one from your IT department can confirm the action
   - Changes were made during non-working hours or on weekends
   - The subject user's name is unknown or unexpected to you
   - The target user's name does not follow a usual naming pattern
   
3. If you confirmed that the action was malicious, find out the login details:
   - Copy the Logon ID field from your 4720 / 4732 event
   - Find the corresponding login event with the same Logon ID
   - Refer to the workbooks from the previous task for further analysis

## Sysmon: Process Monitoring

Sysmon is a free tool from the Microsoft Sysinternals suite that became a de facto standard for advanced monitoring in addition to the default system logs.

Here is a side to side comparison:

| Event Code                            | Purpose | Limitations |
| ----------                            | ------- | ----------- |
| 4688 (Security Log: Process Creation) | Log an event every time a new process is launched | Disabled by default. You need to enable it. |
| 1 (Sysmon: Process Creation)          | Replace 4688 event code and provide more advanced fields like process hash and its signature | Sysmon is an external tool not installed by default.| 

### Analyze Process Launch

1. Open Sysmon logs and filter for event ID 1

2. Review the fields from the process and binary info groups. The red flags are:
   - Image is in an uncommon directory like `C:\Temp` or `C:\Users\Public`
   - Process is suspiciously named like `aa.exe` or `jqyvpqldou.exe`
   - Process hash ( MD5 or SHA256) matches as malware on VirusTotal

3. Review the fields from the parent process group. The red flags are:
   - Parent matches red flags from step 2 (suspicious name, path, or hash)
   - Parent is not expected (e.g. Notepad launching some CMD commands)

4. If still in doubt, go up the process tree until you are confident in your verdict:
   - Find the preceding event where ProcessId equals ParentProcessId in your event
   - Analyze it by following steps 2 and 3 (suspicious parent, name, path, or hash)

5. Finally, trace the attack chain by filtering all Security and Sysmon events with the same Logon ID

## Sysmon: Files and Network

## Powershell: Logging Command

## Conclusion
