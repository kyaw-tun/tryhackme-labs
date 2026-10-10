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

## Credentials Discovery

## Username Enumeration and Password Spraying

## Coercion Attacks

## Mitigations

## Conclusion
