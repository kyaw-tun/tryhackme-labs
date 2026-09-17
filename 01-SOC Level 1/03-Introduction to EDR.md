# Introduction to EDR

> TryHackMe — SOC Level 1
> Room: Introduction to EDR

## Overview

Endpoint Detection and Response (EDR) is a security solution designed to monitor, detect, and respond to advanced threats at the endpoint level. This room explain what is EDR, the use of EDR, the pillars of EDR, the differences between EDR and Antivirus, and the functions of EDR (how it works) and its techniques. And there is a small task at the end which gives a glimpse to what normal EDR dashboard looks like and what it shows.

But is it an IDS (or IDPS)? Are they same? The room didn't really specify it. Intrusion Detection System, acts in a same way too. SO, maybe, EDR is like Host-based IDPS rather than network-based.

## 1. What is EDR?

EDR is as mentioned above Endpoint Detection Response. Previously, i didn't know the difference between EDR and SIEM. But by doing this room again, i think i can be sure of what EDR is. It does three things, it shows, it detects, and it responses. It is essential in large companies that take care of important data and as the rise of people working from home, it is becoming essential to small to medium companies too. 

And what i understand is this:
a company issues a laptop to an employee, they track every step you take on that laptop. And that's for protecting the company as well as your files (if you put personal data on it). 

And it has three pillars

### The Three Pillars

- Visibility
- Detection
- Response

### Visibility

EDR collects data from the endpoints. Like the process you run, the files you create, install application, etc (process modification, registry modifications, file and folders modification, user action, etc). EDR collects all of them. And then takes note of them. And as soon as it sees irregular behavior or anomaly it detects it, and that's the next pillar.  

### Detection

As previously said above, when anomaly occurs, EDR will detect it. And it has more detection capabilities than the traditional detection apps (software, machines, i don't know). And it uses (incorporates) signature based detection, behavior-based detection, such as unusual user activities (like pattern recognition). And so, if any deviation occurs, it will flag it. And then it will take action.   

### Response

After detecting the anomalies, and deviation from normal, it can give the analysts to take action too (which is called response). EDR will show full fledged details on when, where and what happened for you to take the best possible action. You can isolate the endpoint, terminate some process or quarantine some files. And you can connect to the host remotely and execute actions too. 

## 2. EDR vs Antivirus

The course gives the Airport example for this task. Traditional Antivirus is like an Immigration check point. It can only detect known criminals on their database, whereas EDR is like security officers and CCTV cameras, they can detect unusual behavior of certain individuals at the airport, and detect if someone is wandering around restricted areas. Or someone is leaving their bags unattended.

## 3. How EDR Works

### EDR Agent

For multiple endpoints, there are EDR Agents deployed inside them. And these agents are referred to as sensors. They act like eyes and ears to the EDR. Like nosy neighbors, they sit at the endpoint and monitor all the activities and then send them to the console. These agents can do basic signature detection works by themselves and send them to the EDR console with triggered alerts.     

### EDR Console

I don't really know what to say about EDR console. What it does is, it collects data from the agents and then those data is matched with the threat intelligence and then connects the dots. And these dots form a detection which is an alert.

After detection, analyst can acknowledge the alerts and prioritize them. And EDR itself can help and prioritizing them as they can sort the alerts by severity (critical, high, medium, low, informational).

And EDR works with other tools to form a larger security ecosystem. Like Firewalls, DLPs, Email Security Gateways, IAMs, EDRs, and other security solutions protecting the different components of the network. And from what i understand, those all things combine to become a SIEM solution. 

## 4. EDR Telemetry

The data collected by the EDR agents to send to the EDR console is called EDR telemetry. The telemetry data can include regular activity and malicious activity. More data collection leads to better judgment. These data can include:

- Process execution and termination
- Network connections
- Command-line activity
- File/folder modifications
- Registry modifications

## 5. Detection & Response Capabilities

### Detection

- Behavioral Detection - Detecting behavior of a file (e.g. a Microsoft word process `winword.exe` spawning a `powershell.exe`, that's an unusual behavior)
- Anomaly Detection - Deviation from the baseline behavior is an anomaly
- IOC Matching - Detection of any flag that matches known IOC signatures. (and also, IOC = Indicator of compromise)  
- MITRE ATT&CK Mapping - Any activity flagged by EDR is also mapped with the MITRE ATT&CK's tactics and techniques. 
- Machine Learning - Modern EDRs have machine learning models trained by a large dataset of normal and malicious behaviors and can detect complex patterns of attacks.

### Response

- Host isolation
- Process termination
- Quarantine
- Remote access
- Artifact collection

## 6. Investigating an EDR Alert

> Scenario
> Questions and answers


### Investigation Notes

- **Host:** ...
- **Suspicious process:** ...
- **Downloaded payload:** ...
- **Network activity:** ...

## To Conclude
