# CTF Rooms

Capture-the-flag write-ups from TryHackMe. I document the ones I find interesting or that taught me something, rather than every room I complete. Flags are not published.

| Room | Difficulty | Key techniques |
| ---- | ---------- | -------------- |
| [Searchlight - IMINT](./01-Searchlight%20IMINT.md) | Easy | IMINT/GEOINT: image geolocation, reverse image search, Google dorking, video frame analysis |
| [Chocolate Factory](./02-Chocolate%20Factory.md) | Easy | Web foothold, reverse shell, world-readable SSH key, sudo/GTFOBins (vi) privilege escalation, steghide, hashcat |
| [Red (redisl33t)](./03-Red.md) | Easy | LFI filter bypass (readfile, string-replace collapse), hashcat rule-based wordlist, Hydra SSH, /etc/hosts hijack for reverse shell, PwnKit (CVE-2021-4034) |
| [Archangel](./04-Archangel.md) | Easy | Virtual host enumeration, filtered LFI with php://filter, log poisoning to RCE, writable cron script, SUID PATH hijacking |
| [Dogcat](./05-Dogcat.md) | Medium | LFI filter bypass (dog/cat string check, ext= override), php://filter source read, log poisoning to RCE, sudo env (GTFOBins) root, Docker container escape via writable backup script |
| [CyberCrafted](./06-CyberCrafted.md) | Medium | Subdomain enumeration (ffuf), SQL injection (sqlmap), hash cracking, web command execution to reverse shell, readable SSH key, plaintext creds in logs, sudo screen privilege escalation |
| [Juicy Details](./07-Juicy%20Details.md) | Easy | Log analysis / breach investigation across web, SSH, and FTP logs; attack reconstruction and detection notes |

## Concept write-ups

- [Local File Inclusion](./concepts-and-notes/Local-File-Inclusion.md) — an explanation of LFI techniques (filter bypass, php://filter, log poisoning) drawn from the Red, Archangel, and Dogcat rooms above.
- [Linux Privilege Escalation](./concepts-and-notes/Linux-Privilege-Escalation.md) — eight privilege-escalation techniques (kernel exploits, sudo, SUID, capabilities, cron, PATH, NFS) with worked examples.
- [Wifi-Hacking-101](./concepts-and-notes/Wifi-Hacking-101.md) — WPA2 handshake capture and cracking on my own network (monitor mode, airodump-ng, deauthentication, aircrack-ng/hashcat), based on the TryHackMe [Wifi Hacking 101](https://tryhackme.com/room/wifihacking101) and [Wifi Fundamentals and Recon](https://tryhackme.com/room/wififundamentalsandrecon) rooms.
