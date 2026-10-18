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

## Security Log: User Management

## Sysmon: Process Monitoring

## Sysmon: Files and Network

## Powershell: Logging Command

## Conclusion
