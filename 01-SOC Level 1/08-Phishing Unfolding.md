# Phishing Unfolding

A SOC-simulator room from TryHackMe’s SOC Level 1 path. Alerts arrive in real time and you investigate and classify each one as a true or false positive while the attack unfolds, using a SIEM (Splunk), a threat-intelligence tool, and an analyst VM. The scenario completes once you have handled 24 alerts, though more than 30 are generated in total, so no two runs cover exactly the same set.

Two things shaped how I worked this room. The Splunk alerts I pulled didn’t line up with the events in the scenario, so I could not rely on the SIEM to confirm my calls. And when I scanned the suspicious domain in the threat-intelligence tool, it came back clean. So rather than leaning on tooling to tell me what was malicious, I classified alerts on the behavior they described. This write-up is about that reasoning, not a full answer key, so I have kept it to the alerts and indicators that mattered and left the room’s specific data out where I can.

## How I classified the alerts

My 24 alerts were a mix of phishing-email alerts and endpoint (Sysmon) process alerts. For each one I looked at the same things: which host and user it involved, whether the process relationships made sense, where files were being read from or written to, and whether the network activity pointed anywhere it should not. Where threat intel and Splunk didn’t give me a clear answer, the host behavior did.

## The attack chain

The real intrusion ran on the host belonging to the company’s CEO (`michael.ascot`). The clearest signals:

- Unusual parent-child process relationships. PowerShell spawning `nslookup.exe` is not normal user activity. `nslookup` is run from the command line or used internally by Windows, so PowerShell launching it points to a script, and in this case to data being pushed out over DNS.
- DNS exfiltration. Repeated `nslookup` queries were being made to attacker-controlled domains ending in `haz4rdw4re.io`. The calls came from more than one location on the host (both the user’s `Downloads` folder and the `exfiltration` subfolder beneath it), but all pointed at the same domain infrastructure, consistent with data being chunked and pushed out over DNS to the attacker’s server.
- Data staged for theft. The activity ran out of a working directory named `C:\Users\michael.ascot\Downloads\exfiltration\`, and sensitive files (financial records) were being read and copied before being sent out.

Taken together: initial access through phishing, then execution via PowerShell, then collection of sensitive files, then exfiltration over DNS, all centered on the CEO’s machine.

## Where I didn’t rely on the tools

Two points worth being honest about, because they’re the actual lesson of this room for me:

- Splunk didn’t confirm the scenario events. The alerts I found there didn’t correspond to what the scenario was showing me, so I classified from the host-level indicators in each alert instead of waiting for the SIEM to validate them.
- The malicious domain scanned clean. `*.haz4rdw4re.io` came back clean in the threat-intel tool, but the behavior around it (PowerShell-driven `nslookup`, encoded queries, an `exfiltration` folder) was clearly malicious. A clean reputation score doesn’t mean safe, since new attacker infrastructure often is not flagged yet. I classified on what the activity was doing, not on what the tool said about the domain.

## Where I miscalibrated

On my run I classified 9 alerts incorrectly, and they were all on the phishing-email side: I marked benign emails as threats. The endpoint and process alerts, which made up the actual attack chain, I classified correctly. After completing the scenario I went back through those 9 specifically to understand why I’d misread them, and the pattern was consistent: I was flagging emails on how suspicious they looked rather than on concrete indicators. Over-flagging is the safer direction to err, but it still costs analyst time, so the lesson for me is to judge email the same way I judged the endpoint alerts, on evidence rather than tone.

## Takeaway

The most reliable detection point in this chain was not the email or the domain reputation, it was the behavior: PowerShell spawning `nslookup` to reach an external domain, running out of a directory named for exfiltration. That parent-child relationship is hard for an attacker to disguise, which makes it a stronger thing to alert on than reputation data that may not have caught up yet.
