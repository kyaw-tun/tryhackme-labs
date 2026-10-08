# AD Basic Enumeration

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [AD Basic Enumeration](https://tryhackme.com/room/adbasicenumeration)

Active Directory is the phrase I started hearing when I started learning cybersecurity concepts many months ago. As a Linux user for most of my life, I was not familiar with it, nor have I encountered it back when I was on Microsoft Windows. Even though I now know the importance of it, I can't find a Linux Equivalent to it (Maybe there isn't one).

This room introduce basic enumeration steps to Microsoft active directory using tools such as `fping`, `nmap`, in mapping out the network, `smbclient` in network enumeration, `ldapsearch`, `rpcclient`, `kerbrute` in domain enumeration, and `crackmapexec` in password spraying section.  

## Mapping out the Network

This section is just like using nmap on targets. And you can use nmap for this. The room instructor used `fping` and `nmap`.

Here is how you can discover live hosts within the network using fping:

```bash
fping -agq 10.211.11.0/24
```
- `a` - shows system that are alive
- `g` - generates a target list from a supplied IP netmask.
- `q` - quiet mode, doesn't show per-probe results or ICMP error messages.

Alternatively, you can discover live hosts with nmap like this:

```bash
nmap -sn 10.211.11.0/24
```
- `sc` - Ping scan

After we find the hosts that are live, we can map which ports are open. This is the basic port scanning on nmap. For this, you can add the discovered ip addresses from earlier into a `.txt` file, or you can do it by mentioning all the ip addresses on the nmap scan.

Since this is the Active Directory room, we will scan these ports specifically because they are the ones mostly associated with it. 

| Port | Protocol | What it means |
| --- | --- | --- |
| 88 | Kerberos | Potential for Kerberos-based enumeration |
| 135 | MS-RPC | Potential for RPC enumeration (null sessions) |
| 139 | SMB/NetBIOS | Legacy SMB access |
| 389 | LDAP | LDAP queries to AD |
| 445 | SMB | Modern SMB access, critical for enumeration |
| 464 | Kerberos (kpasswd) | Password-related Kerberos service |

So, for all those ports, nmap command would be:

```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
```
Here, I am calling the IP addresses from an external `.txt` file. And

- `iL` means reading from an external file, in this case `hosts.txt`.

And another command mentioned in the room is doing syn scan. The command goes like this:

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```
Here: 
- `-sS` - means TCP syn scan
- `-p-` - means all TCP ports not just first 1000 ports that nmap scans on default
- `-T3` - Sets the timing template to "normal" to balance speed and stealth.

However, my nmap with a version of 7.99 refuses to scan all the ports, or even do `-A`. And not just this command, I have encountered in many other rooms too. But previous versions of nmap like the one on Attackbox, with a version of 7.80, can run this command. 

## Network Enumeration with SMB

Now that we can see which ports are open and which ones are not, we will have to figure out the next step. Here is the output from nmap:

```bash
[...]

Nmap scan report for 10.211.11.10
Host is up (0.39s latency).

PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-10-08 02:23:23Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc, Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
[...]

```
We still don't have anything yet but we can check if we can access port 139. I think of this like `ftp` port, sometimes, it can be accessed anonymously. For this, we can use `smbclient`, or `smbmap`.

And we can access the port 139 by this command anonymously (without needing credentials):

```bash
smbclient -L //10.211.11.10 -N
```

And turns out there are some shares that can has read and write permission for anonymous users namely: `AnonShare`, `SharedFiles`, `UserBackups`. And we can access those shares by:

```bash
smbclient //10.211.11.10/SharedFiles -N
```

If we were to have username and password, you can replace `-N` with `--user=USERNAME --password=PASSWORD` or `-U 'username%password'`.

You can also utilize other tools for this such as: `impacket-smbclient`, `crackmapexec` and `enum4linux`, etc. And we can even use our own `nmap` for this too using its `smb-enum-shares` script.

## Domain Enumeration

### LDAP Enumeration

LDAP means lightweight Directory Access Protocol, runs on the port 389, widely used protocol for accessing and managing directory services, such as Microsoft Active Directory. 

We can test if anonymous LDAP bind is enabled with `ldapsearch`:

```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```
- `-x` - Simple authentication
- `-H` - LDAP server
- `-s` - Limits the query only to the base object 

Once we know we can, we can get user information with this command:

```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)" 
```
### RPC Enumeration

Microsoft Remote Procedure Call (MSRPC) is a protocol that runs on the port 135 and enables a program running on one computer to request services from a program on another computer, without needing to understand the underlying details of the network.

We can verify null session with:

```bash
rpcclient -U "" 10.211.11.10 -N
```

- `-U` means username but we are using "" because we are logging in anonymously
- `-N` means no passwords

Once we are in, we can enumerated user names with `enumdomusers`.

### RID cycling

We can manually try querying each individual user RID if `enumdomusers` is restricted with this command:

```bash
for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
```

However, when I did this, I was only able to enumerate the first 3 users from when I ran with `rpcclient`. And I did try multiple times.   

## Password Spraying

## Conclusion
