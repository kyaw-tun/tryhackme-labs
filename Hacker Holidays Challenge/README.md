# Hacker Holidays Challenge

Write-ups from TryHackMe's Hacker Holidays Challenge (Byte Lotus), which I completed in full. I document a couple of the more interesting challenges here rather than all of them.

| Challenge | Topic | Key techniques |
| --------- | ----- | -------------- |
| [Room 404](01-Room%20404.md) | Web, exposed version control data | Directory enumeration with `gobuster`, manual analysis of exposed repository objects with `python3` |
| [Complimentary](02-Complimentary.md) | Cloud security (AWS) | Credential discovery in client-side code and network traffic, AWS CLI configuration, DynamoDB table scan |
| [Beach Bar](03-Beach%20Bar.md) | Web exploitation and privilege escalation | Source code review, unsafe YAML deserialization for RCE, reverse shell, service enumeration with `systemctl` |
| [Do Not Disturb](04-Do%20Not%20Disturb.md) | Web, NoSQL injection and server-side template injection | NoSQL injection auth bypass, cookie reuse, SSTI (EJS) to RCE, Node.js inspector (`node inspect`) lateral movement, privilege escalation via `disk` group and `/etc/shadow` cracking |

*More write-ups in progress.*
