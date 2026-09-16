# Introduction to SIEM

> TryHackMe — SOC Level 1
> Room: Introduction to SIEM

## Overview

Brief introduction to what SIEM is and what this room covers.

---

## 1. What is SIEM?

### SIEM Definition

...

### Why SOC Analysts Use SIEM

...

---

## 2. Logs Everywhere, Answers Nowhere

### Host-Centric Log Sources

- Windows
- Linux
- Servers
- Other endpoint activity

### Network-Centric Log Sources

- Firewalls
- IDS/IPS
- Routers
- VPN
- Network services

### Challenges of Isolated Logs

- Numerous log sources
- No centralization
- Limited context
- Limited analysis
- Different log formats

---

## 3. Why SIEM?

### Centralized Log Collection

...

### Log Normalization

...

### Log Correlation

...

### Real-Time Alerting

...

### Dashboards & Reporting

...

### Other SIEM Capabilities

- Threat intelligence
- Data retention
- Search capabilities
- ...

---

## 4. Log Sources & Ingestion

### Windows

- Event Viewer
- Windows Event IDs
- Examples of useful events

### Linux

Important log locations:

- `/var/log/httpd`
- `/var/log/cron`
- `/var/log/auth.log`
- `/var/log/secure`
- `/var/log/kern`

### Web Server

- Apache logs
- HTTP requests/responses
- Common log locations

### Log Ingestion Methods

1. Agent / Forwarder
2. Syslog
3. Manual Upload
4. Port Forwarding

---

## 5. Alerting & Analysis

### Detection Rules

What are detection rules?

...

### Example: Event Log Cleared

**Condition:**

...

**Alert:**

...

### Example: `whoami` Execution

**Relevant fields:**

- Log source
- Event ID
- NewProcessName

**Condition:**

...

**Alert:**

...

### Alert Investigation

1. Alert triggered
2. Examine associated events
3. Check which rule was triggered
4. Determine True Positive / False Positive
5. Take appropriate action

### Possible Responses

- Tune detection rule
- Investigate further
- Contact asset owner
- Isolate host
- Block suspicious IP

---

## 6. Lab Investigation

### Scenario

Briefly describe the suspicious activity scenario.

### Investigation

**Triggered process:**
...

**User responsible:**
...

**Hostname:**
...

**Detection rule:**
...

**Matched term:**
...

### Verdict

**False Positive / True Positive**

...

### Response

...

### Flag

`...`

---

## Conclusion

- SIEM provides centralized visibility across many log sources.
- ...
- ...
- ...
A short paragraph about what you personally learned
from the room and how SIEM fits into SOC operations.
