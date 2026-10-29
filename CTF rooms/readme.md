# CTF Rooms

Capture-the-flag write-ups from TryHackMe. I document the ones I find interesting or that taught me something, rather than every room I complete. Flags are not published.

| Room | Difficulty | Key techniques |
| ---- | ---------- | -------------- |
| [Searchlight - IMINT](./01-Searchlight%20IMINT.md) | Easy | IMINT/GEOINT: image geolocation, reverse image search, Google dorking, video frame analysis |
| [Chocolate Factory](./02-Chocolate%20Factory.md) | Easy | Web foothold, reverse shell, world-readable SSH key, sudo/GTFOBins (vi) privilege escalation, steghide, hashcat |
| [Red (redisl33t)](./03-Red.md) | Easy | LFI filter bypass (readfile, string-replace collapse), hashcat rule-based wordlist, Hydra SSH, /etc/hosts hijack for reverse shell, PwnKit (CVE-2021-4034) |
| [Archangel](./04-Archangel.md) | Easy | Virtual host enumeration, filtered LFI with php://filter, log poisoning to RCE, writable cron script, SUID PATH hijacking |
