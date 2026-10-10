# Intro to AD Breaching

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Intro to AD Breaching](https://tryhackme.com/room/introductiontoactivedirectorybreaching)
> 

## Introduction

By now, I have covered basic Enumeration and authentication in Active Directory. But in real world scenario, to do that, you need set of valid credentials. Without them, you won't be able to enumerate anything in AD nor move laterally and certainly won't be able to escalate privileges. The process of obtaining those initial credentials is called breaching.

First off, the rooms asked you to add multiple IP addresses as with their domain names in `/etc/hosts`. The WebServer hosts three services behind an Nginx reverse proxy, each accessible via its own hostname: `git.thm.loc`, `ci.thm.loc`, and `printer.thm.loc`.

## Active Directory Breaches

## OSINT And Target Reconnaissance

## Credentials Discovery

## Username Enumeration and Password Spraying

## Coercion Attacks

## Mitigations

## Conclusion
