# Introduction to SIEM

**TryHackMe Room:** [Introduction to SIEM](https://tryhackme.com/room/introtosiem)

This write-up covers my understanding of the room and the key concepts I took away from it.

## Overview

This room introduces Security Information and Event Management (SIEM), including what a SIEM is, where its data comes from, how logs are collected and processed, and how detection rules can be used to identify suspicious activity.

The room also includes a small investigation lab where an alert is triggered and the analyst has to investigate the associated events and determine whether the activity is a true positive or a false positive.

## 1. What is SIEM?

### SIEM Definition

SIEM stands for Security Information and Event Management.

A SIEM is one of the core security solutions used by Security Operations Center (SOC) analysts. It collects security-related data from different sources, centralizes it, and provides tools for searching, analyzing, correlating, and alerting on that data.

The main idea is to take security events that would otherwise be scattered across many different systems and make them available in one place for analysts to investigate.

## 2. Logs Everywhere, Answers Nowhere

Almost every system generates logs when something happens.

For example, a system might record events when:

- A user creates or deletes a file.
- A process is executed.
- A registry entry is modified.  
- A user attempts to authenticate.
- A device communicates with another system.
- A user connects through a VPN.
- Network traffic passes through a firewall.

These events can come from many different sources, and the resulting logs can vary significantly in format and structure.

Two useful categories are host-centric log sources and network-centric log sources.

### Host-Centric Log Sources

Host-centric logs describe activity occurring on individual systems or endpoints.

Examples include:

- File creation or deletion
- PowerShell or other process execution
- Registry modifications
- Authentication attempts
- Other endpoint activity

These logs can come from systems such as:

- Windows 
- Linux
- Servers
- Other endpoint activity

### Network-Centric Log Sources

Network-centric logs describe activity involving communication between systems or interaction with network services.

Examples include:

- SSH connections
- FTP activity
- Web traffic
- VPN connections
- Access to network resources

These logs can come from sources such as:

- Firewalls
- IDS/IPS
- Routers
- VPN
- Network services

### Challenges of Isolated Logs (Answer Nowhere)

Having many independent log sources creates several problems for security analysts:

- Numerous log sources 
- No centralization
- Limited context
- Limited analysis
- Different log formats

For example, an analyst investigating a suspicious login might need to look at authentication logs, firewall logs, endpoint activity, and other sources separately. Without centralization and correlation, connecting these events together becomes much more difficult.

## 3. Why SIEM?

This is where SIEM becomes useful.

Instead of having security logs scattered across firewalls, servers, workstations, and other infrastructure, a SIEM can collect those logs into a centralized platform.

It can also normalize the data and correlate events from different sources, making it easier for analysts to understand what is happening across the environment.

A SIEM does not necessarily detect everything automatically. Detection rules and other detection logic need to be configured so that the SIEM knows what activity should generate an alert.

### Core SIEM features

### Centralized Log Collection

A SIEM collects logs from different systems and makes them available from a central location.

This can include logs from:

- Firewalls
- Servers
- Workstations
- Network devices
- Security tools

### Log Normalization

Different systems can produce logs in different formats.

Log normalization transforms this data into a more consistent structure, making it easier to search, analyze, and correlate.

### Log Correlation

Correlation involves connecting events from different sources and identifying relationships between them within a particular time period.

For example, multiple failed login attempts followed by a successful login could be more significant when considered together than when each event is viewed independently.

### Real-Time Alerting

A SIEM can evaluate incoming events against configured detection rules.

When a rule is triggered, the SIEM can generate an alert for an analyst to investigate.

### Dashboards and Reporting

SIEM platforms commonly provide dashboards that allow analysts to visualize events, alerts, and other security information.

They can also provide reporting capabilities for reviewing security activity.

### Other SIEM Capabilities

The room also briefly mentioned several other capabilities that can be found in SIEM solutions:
 
- Threat intelligence
- Data retention
- Search capabilities

## 4. Log Sources & Ingestion 

A SIEM needs a way to receive logs from the systems it monitors. The exact implementation depends on the SIEM and the environment.

### Windows

Windows provides event logs that can be viewed using Event Viewer.

These logs contain information about activities occurring on the system, such as authentication events, process creation, system events, and other security-related activity.

### Linux

Linux systems commonly store logs under `/var/log/`, although the exact files and directories can vary depending on the distribution and services installed.

Some examples include:

- `/var/log/httpd`  — HTTP server logs on systems using Apache/httpd
- `/var/log/cron` — cron-related activity
- `/var/log/auth.log` — authentication-related logs on Debian-based systems
- `/var/log/secure` — authentication-related logs on some Red Hat-based systems
- `/var/log/kern` — kernel-related logs on some Linux distributions

Not every Linux system will contain all of these files. Their availability depends on the distribution, logging configuration, and services installed.

### Web Server

Web servers also generate logs that can be collected by a SIEM.

For Apache, common locations include: `/var/log/apache2` && `/var/log/httpd`.

The exact location depends on the operating system and configuration.

### Log Ingestion Methods

Different SIEM solutions support different methods of collecting logs. Some common approaches include:

1. Agent / Forwarder 

An agent is installed on an endpoint and collects relevant logs before forwarding them to the SIEM.

This is somewhat similar to the agent-and-console architecture used by EDR solutions.

2. Syslog

Syslog is a common logging protocol used to send event data from systems and network devices to a centralized destination.

3. Manual Upload

Some SIEM platforms allow analysts to upload log files manually for analysis. This can be useful for offline investigation or analyzing previously collected data.

4. Port Forwarding 

A SIEM can listen on a designated port while an endpoint or other system forwards log data to that destination.

## 5. Alerting & Analysis

How does a SIEM trigger an alert?

A SIEM can use detection rules to identify activity that matches a particular condition.

For example, a rule could look for:

- Five failed login attempts within ten seconds
- A successful login immediately following multiple failed attempts
- Unauthorized USB device activity
- Outbound network traffic exceeding a particular threshold

When the configured conditions are met, the SIEM can generate an alert for an analyst to investigate.

### Detection Rules

Detection rules define the conditions that should cause a SIEM to generate an alert.

The exact syntax depends on the SIEM, but the general idea is to take observable events and define conditions that represent suspicious or otherwise important activity.

### Example: Event Log Cleared

Attackers may attempt to clear logs to remove evidence of their activity.

Windows Event ID 104 can indicate that an event log was cleared.

A simple detection rule could therefore be:

> **Rule**: If Log Source is WinEventLog **AND** EventCode is **4688**, and NewProcessName contains **whoami**, then Trigger an ALERT `WHOAMI command Execution DETECTED`

This does not necessarily mean that an attacker is responsible every time the rule triggers. An analyst would still need to investigate the event and determine whether it represents legitimate activity or malicious behavior.

### Example: `whoami` Execution

The `whoami` command can be useful to an attacker after gaining access to a system because it reveals the current user context.

Windows process creation events can be used to detect command execution. For example, Event ID 4688 records process creation.

A detection rule could therefore look for:

> **Rule**: If Log Source is WinEventLog **AND** EventCode is **4688**, and NewProcessName contains **whoami**, then Trigger an ALERT `WHOAMI command Execution DETECTED`

Again, detecting whoami alone does not prove that malicious activity occurred. Legitimate administrators and users may also execute the command.

### Alert Investigation

When an alert is triggered, the analyst needs to investigate the associated events and determine why the detection rule fired.

The analyst then needs to classify the activity as either:

- False Positive — the alert was triggered, but the activity was legitimate or otherwise not malicious.
- True Positive — the alert correctly identified suspicious or malicious activity.

The appropriate response depends on the results of the investigation and the organization's procedures.

### Possible Responses

Depending on the situation, possible actions include:

- False Positive: Tune or adjust the detection rule to reduce unnecessary alerts.
- True Positive: Continue investigating the incident and determine its scope and impact.
- Contact the asset owner: Ask the owner whether the observed activity was expected or authorized.
- Isolate the host: If an endpoint is suspected to be compromised, isolate it from the network to limit further activity.
- Block a suspicious IP: Block communication with a known malicious or suspicious destination when appropriate.

The exact response should depend on the evidence, severity, affected systems, and the organization's incident-response procedures.

## 6. Lab Investigation

I won't go into detail about the lab itself because it was relatively straightforward.

The lab first presents a normal SIEM dashboard. After starting the simulated suspicious activity, an alert is generated.

The task is then to investigate the alert, examine the available information, determine whether the activity is a true or false positive, and identify the flag.

## Conclusion

After redoing the EDR room yesterday and SIEM today, I feel like I have a much clearer understanding of the difference between the two.

EDR is focused primarily on monitoring, detecting, investigating, and responding to activity on endpoints, while SIEM provides a centralized place to collect and analyze security data from many different sources.

Doing these two rooms one after another helped connect those concepts for me. Rather than seeing them as two unrelated security tools, I can now better understand how they can provide different types of visibility to a SOC.
