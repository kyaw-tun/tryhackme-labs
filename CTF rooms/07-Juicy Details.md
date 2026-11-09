# Juicy Details

> **TryHackMe Room**: [Juicy Details](https://tryhackme.com/room/juicydetails)

A log-analysis investigation. I was given three log files (`access.log`, `auth.log`, `vsftpd.log`) and had to reconstruct a web attack by reading them. This write-up covers how I worked through the logs and the reasoning I used. Answers and flags are not included.

## The logs
- `access.log` — records web requests to the server; the attacker's tools, scans, and payloads show up here.
- `auth.log` — records SSH authentication attempts.
- `vsftpd.log` — records FTP activity.

## Reconstructing the attack

### Reconnaissance and web attack (access.log)

Reading access.log, the attacker's early activity followed a familiar boot2root pattern: port/endpoint discovery, directory enumeration, automated SQL injection testing, and a brute-force attack against a login endpoint. Because the tools were used one at a time, each left a distinct, easily identifiable burst in the log (recognizable by the request patterns and user-agents).

A few things I worked through here:
- Identifying which endpoint was hit by the brute-force vs. which was tested for SQL injection, by matching each tool's activity to the URL it targeted.
- The SQL injection *parameter*: this tripped me up at first, because I misread "parameter" as the injection characters (`'`, `"`, `)`) rather than the query parameter name (e.g. the `name` in `/path/?name=...`). Once I understood the term, it was straightforward. The payloads were URL-encoded, so I had to decode them to read what was being tested.

### Data access (SQL injection and file retrieval)

Toward the end of access.log, the later requests were shorter and unencoded, making the final stages easier to read. From these I could see the attacker move from testing the SQL injection to actually pulling user data from the vulnerable endpoint, and then attempting to retrieve files from another endpoint.

### Credential access and exfiltration (vsftpd.log and auth.log)

Cross-referencing the other two logs filled in the rest:
- `vsftpd.log` showed the FTP service being used, including the account it authenticated as, to retrieve files.
- `auth.log` showed a separate brute-force attempt, this time against SSH, and the service/username used to get shell access.

### A note on the brute-force "success"

One question asked whether the web brute-force succeeded. I initially assumed not, since I saw only `401`/`500` responses and no obvious success code. That was the wrong read, and it taught me not to rely only on a single "success" status when judging whether an attack worked; the timing and sequence of requests matter too.

## What a defender would catch

Almost every stage of this attack was visible in the logs if someone was watching:
- Automated tools leave obvious fingerprints (request bursts, tool user-agents) — easy to alert on.
- A run of failed logins against one endpoint, followed by changed behavior, is a brute-force signature.
- SQL injection attempts show up as encoded payloads in request parameters.
- FTP and SSH authentications in auth/vsftpd logs tie the web attack to actual account compromise.
The takeaway: these three logs together told the whole story, which is exactly why centralized logging and monitoring matter.

## Conclusion

This room is approachable, especially with some `boot2root` CTF experience, but it is really a test of reading logs carefully and understanding what each request means. My main lesson was a simple but important one: read the question (and the evidence) precisely. I misread "parameter" and misjudged the brute-force result at first, both from assuming rather than checking. 

One honest habit I want to drop: I sometimes use the answer-field length (the number of characters expected) to narrow down what I am looking for. It helps in a CTF, but it is a crutch that would not exist in real investigations, so it is worth training myself off it.
