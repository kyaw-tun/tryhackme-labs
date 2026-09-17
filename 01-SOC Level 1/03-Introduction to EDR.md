# Introduction to EDR

> TryHackMe — SOC Level 1
> Room: Introduction to EDR

## Overview

Endpoint Detection and Response (EDR) is a security solution designed to monitor, detect, investigate, and respond to threats at the endpoint level.

This room covers what EDR is, its main uses and three pillars, the differences between EDR and traditional antivirus, how EDR works, and some of its detection and response capabilities. At the end, there is also a small investigation task that gives a glimpse of what a typical EDR dashboard looks like and the type of information it provides.

One question I had while going through the room was whether EDR is basically an IDS/IDPS for endpoints. They are similar in the sense that both monitor activity and look for suspicious behavior, but they operate in different areas. An EDR focuses specifically on endpoints, while an IDS/IPS can also monitor network traffic and other parts of an environment. So, it is reasonable to think of EDR as having some similarities to a host-based detection and response system, although EDR provides capabilities beyond simply detecting intrusions.

## 1. What is EDR?

EDR stands for Endpoint Detection and Response.

Before going through this room, I did not have a clear understanding of the difference between EDR and SIEM. After going through it, I have a better idea of what EDR actually does.

At a high level, EDR provides three main things:

- Visibility
- Detection
- Response

EDR is especially important for organizations that handle sensitive or valuable data. With more people working remotely, protecting endpoints such as company laptops has also become increasingly important.

One simple way I understand EDR is this:

Imagine a company gives an employee a laptop. An EDR solution installed on that laptop monitors what is happening on the endpoint, such as processes being executed, files being modified, applications being installed, and network connections being made.

The purpose is not simply to "track the employee." The collected information allows the security team to identify potentially malicious activity and investigate what happened if something goes wrong.

### The Three Pillars

### Visibility

EDR collects information from endpoints. This can include processes being executed, files being created or modified, applications being installed, registry changes, network connections, and other user or system activity.

The EDR keeps a record of this activity so that it can be monitored and investigated later.

Having this visibility is important because the EDR needs to know what is happening on an endpoint before it can identify something unusual. 

### Detection

Once EDR has visibility into endpoint activity, it can look for suspicious or unusual behavior.

EDR can use multiple detection methods, including signature-based detection, behavior-based detection, and anomaly detection. For example, an unusual sequence of processes or activity that deviates from normal behavior could be flagged for investigation.

When suspicious activity is detected, the EDR generates an alert that can then be investigated by a security analyst. 

### Response

After suspicious activity is detected, EDR provides tools that can help analysts respond to it.

An EDR alert can provide details about what happened, when it happened, which endpoint was involved, and what processes or files were associated with the activity.

Depending on the EDR solution, an analyst may be able to:

- Isolate the endpoint from the network
- Terminate a suspicious process
- Quarantine a malicious file
- Connect to the endpoint remotely
- Collect artifacts for further investigation

This allows analysts to not only detect an incident, but also take action against it.

## 2. EDR vs Antivirus

The room uses an airport example to explain the difference between traditional antivirus and EDR.

Traditional antivirus can be thought of as an immigration checkpoint. It can check people against known records and identify known threats.

EDR is more like having security officers and CCTV cameras throughout the airport. Instead of only looking for known threats, it can observe behavior and identify suspicious activity.

For example, security officers might notice someone repeatedly entering restricted areas or leaving a bag unattended. They may not know that person is a criminal, but the behavior itself is suspicious enough to investigate.

This is similar to how EDR can identify suspicious behavior even when there is no known signature for the threat.

The main difference I took from the room is that traditional antivirus primarily focuses on preventing and detecting known malicious software, while EDR provides much more visibility into what is happening on an endpoint and gives analysts additional capabilities for investigation and response.

## 3. How EDR Works

### EDR Agent

For environments with many endpoints, an EDR agent is deployed on each endpoint.

These agents are sometimes referred to as sensors. They act as the eyes and ears of the EDR system.

They sit on the endpoint and monitor activities such as process execution, file changes, network connections, and other system activity. The collected information is then sent to the EDR console for analysis.

Some EDR agents can also perform basic detection locally and generate alerts when they identify suspicious activity.

I like to think of the agents as nosy neighbors: they sit on the endpoint and keep an eye on what is happening, then report interesting activity back to the EDR console.  

### EDR Console

The EDR console is where the collected endpoint data is brought together.

Data from the EDR agents is analyzed and can be correlated with threat intelligence and other security information. By connecting different pieces of activity together, the EDR can identify suspicious behavior and generate alerts.

Analysts can then investigate and prioritize these alerts. EDR platforms commonly allow alerts to be sorted by severity, such as:

- Critical
- High
- Medium
- Low
- Informational

EDR can also work alongside other security solutions, such as firewalls, DLP, email security gateways, and IAM systems.

This is where EDR differs from a SIEM. EDR is focused primarily on endpoint visibility, detection, investigation, and response, while a SIEM collects and correlates logs and security events from many different sources across an environment.

## 4. EDR Telemetry

The data collected by EDR agents and sent to the EDR platform is known as EDR telemetry.

Telemetry can contain both normal and potentially malicious activity. Having more useful telemetry generally gives analysts more information to work with when investigating an alert.

Examples of telemetry include:

- Process execution and termination
- Network connections
- Command-line activity
- File and folder modifications
- Registry modifications

This telemetry is important because it gives analysts the context needed to understand what happened on an endpoint.

## 5. Detection & Response Capabilities

### Detection

EDR can use several different techniques to identify suspicious activity.

- Behavioral Detection — Detects suspicious behavior rather than relying only on a known signature. For example, a Microsoft Word process `winword.exe` spawning PowerShell `powershell.exe` could be considered suspicious depending on the context.

- Anomaly Detection  — Identifies activity that deviates from an established baseline or expected behavior.

- IOC Matching — Looks for known Indicators of Compromise (IOCs), such as known malicious IP addresses, domains, file hashes, or other artifacts.

- MITRE ATT&CK Mapping — EDR alerts can be mapped to techniques and tactics from the MITRE ATT&CK framework. This helps analysts understand what part of an attack an observed activity may represent.

- Machine Learning — Modern EDR solutions can use machine learning models trained on large amounts of normal and malicious activity to identify complex patterns that may be difficult to detect using simple signatures.

### Response

Once suspicious activity has been detected, EDR can provide several response capabilities:

- Host isolation — Isolate the affected endpoint from the network.
- Process termination — Stop a malicious or suspicious process.
- Quarantine — Isolate a suspicious or malicious file.
- Remote access — Allow an analyst to remotely access the endpoint for investigation or response.
- Artifact collection — Collect relevant files, logs, processes, or other artifacts for further investigation.

## 6. Investigating an EDR Alert

The final task provides a practical look at how an EDR platform can be used to investigate alerts across different endpoints.

The questions are mainly focused on reading the information provided by the EDR and identifying details such as the affected host, suspicious processes, downloaded files, and network activity.

I did not include the individual answers here because they are mostly straightforward lookups from the provided alerts. Instead, the main takeaway for me was understanding what information an analyst would normally look at when investigating an alert:

- Which endpoint generated the alert?
- What process or activity triggered it?
- What files or payloads were involved?
- Was there any related network activity?
- What happened before and after the suspicious activity?

This part of the room was useful because it connected the earlier concepts about EDR telemetry and detection to an actual investigation workflow.

## To Conclude

This room helped me get a better understanding of what EDR actually does and, more importantly, how it differs from other security tools.

The three main ideas I took away are visibility, detection, and response. EDR collects detailed telemetry from endpoints, uses that information to detect suspicious activity, and gives security analysts tools to investigate and respond to it.

I also have a clearer distinction between EDR and SIEM now. EDR focuses on what is happening on endpoints and provides investigation and response capabilities, while SIEM is more focused on collecting and correlating security events from many different sources.

Overall, I think of EDR as a security layer that gives analysts eyes and ears on an endpoint, along with the ability to actually do something when suspicious activity is found.
