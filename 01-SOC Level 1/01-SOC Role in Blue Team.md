# SOC Role in Blue Team

## Introduction

This room introduces the role of a Security Operations Center (SOC) within an organization or enterprise structure.

Most of the concepts in this room were already familiar to me from my previous cybersecurity studies. However, I included it in my portfolio because the final challenge exposed a gap in my understanding of how specific security tasks are divided between different roles.

## Security Hierarchy 

A typical security organization can include several levels of responsibility:

- Executives — such as the Chief Executive Officer (CEO).
- Security Leadership — including the Chief Technology Officer (CTO), Chief Information Officer (CIO), and Chief Information Security Officer (CISO).
- Security Management — including security managers and team leads.
- Technical Roles — including analysts, engineers, incident responders, penetration testers, and other technical specialists.

The exact structure varies between organizations. Smaller companies may not have a dedicated security department, with security responsibilities instead being handled by the IT department.

### Security Departments

Organizations with dedicated security teams may divide responsibilities into different departments:

- Red Team — focuses on offensive security, including penetration testing, ethical hacking, and adversary simulation.
- GRC Team — focuses on governance, risk, security policies, audits, and compliance requirements such as PCI DSS.
- Blue Team — focuses on defensive security, including security monitoring, detection, incident response, and defensive engineering.

## Meet the Blue Team

The Blue Team is responsible for defending an organization's systems and infrastructure. This includes continuously monitoring for suspicious activity and responding to security incidents.

### Security Operations Center (SOC)

A SOC is responsible for security monitoring, alert investigation, and incident handling.

A typical SOC may contain several roles:

- L1 Analysts — perform initial alert triage and investigation and escalate suspicious or more complex cases.
- L2 Analysts — perform deeper investigations and handle incidents requiring additional analysis.
- Security Engineers — build and maintain security infrastructure, detection systems, and related tooling.
- SOC Managers — manage the SOC team, processes, priorities, and overall operations.

### Cyber Incident Response Team (CIRT)

A Cyber Incident Response Team (CIRT), also known as a CSIRT or CERT, is responsible for responding to significant or complex security incidents.

When an incident becomes too serious or complex to be handled through normal SOC procedures, the incident response team can become involved in containment, investigation, and recovery.

Examples of incident response organizations discussed in the room include:

- JPCERT
- Mandiant
- AWS CIRT

### Specialized Defensive Roles

Larger organizations may also have specialized security roles requiring deeper knowledge in specific areas.

Examples include:

- Digital Forensics Analyst
- Threat Intelligence Analyst
- Application Security (AppSec) Engineer
- Security-focused AI researchers
- DevSecOps specialists

These specialists can support SOC and incident response teams when an investigation requires expertise in a particular area.

## Internal SOC vs MSSP

Not every organization operates its own SOC.

An organization can have an internal SOC, where security monitoring and response are handled by employees within the company.

Alternatively, an organization can use a Managed Security Services Provider (MSSP) to provide services such as security monitoring, alert triage, detection, and incident response.

Working for an MSSP can also expose analysts to different environments and a wide range of security incidents, although the work can be fast-paced because analysts may be responsible for multiple customers.

## Final Challenge

The final challenge required assigning different security incidents and tasks to the appropriate security roles.

### Roles

- **Susan** — SOC L2 Analyst
- **Nick** — GRC Auditor
- **Lucas** — SOC L1 Analyst
- **Eugen** — SOC Engineer
- **Robert** — CERT Lead
- **Ben** — Penetration Tester
- **Alice** — Threat Researcher

### My Attempt

I had difficulty with this challenge when I originally completed the room and needed multiple attempts before getting the assignments right.

When revisiting the room for this portfolio, I was able to complete the challenge correctly on my first attempt. I approached it mainly by matching each task to the role whose responsibilities were the most obvious:

| Order | Task | Role |
|---|---|---|
| 1 | Servers storing credit card information require a PCI DSS audit | **Nick — GRC Auditor** |
| 2 | The office in France was hit with ransomware and immediate response was required | **Robert — CERT Lead** |
| 3 | SIEM created an alert about brute-force activity against FW-NY-01 | **Lucas — SOC L1 Analyst** |
| 4 | The SIEM is unavailable because of a storage limit | **Eugen — SOC Engineer** |
| 5 | Check the new version of tryhackme.thm for vulnerabilities | **Ben — Penetration Tester** |
| 6 | FIN7 is actively targeting the company and their tactics need to be analyzed | **Alice — Threat Researcher** |
| 7 | HR manager Anna launched phishing malware and a deep analysis is required | **Susan — SOC L2 Analyst** |

The phishing malware investigation was the one I initially overlooked. I did not immediately associate it with the SOC L2 role and instead eliminated the other obvious choices first. Since the remaining role was the L2 analyst, it became clear that deeper analysis of a phishing/malware incident can also fall under L2 responsibilities.
