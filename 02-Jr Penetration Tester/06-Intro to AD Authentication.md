# Intro to AD Authentication

This write-up covers my understanding of the room and the key concepts I took away from it.

> **TryHackMe Room:** [Intro to AD Authentication](https://tryhackme.com/room/introtoactivedirectoryauthentication)
> 

## Introduction

This room introduces how authentication works in Active Directory environment. Once we gain access to an Active Directory, it is necessary to understand how authentication works. And in this room, two of the primary authentication protocols are discussed: NetNTLM and Kerberos authentications.  

## Authentication in AD

What is authentication? Authentication is the process of proving your identity, essentially answering the question: "Are you who you claim to be?"

### Authentication Materials

Most authentication works over usernames and passwords. But there are some other authentication materials too, such as, certificates, hashes. 

### Authentication vs Authorization

- Authentication asks the question: "Are you who you say you are?"
- Authorization asks the question: "Are you allowed to access this?"

And authentication always comes first.

### AD Authentication Protocols

There are two primary protocols that handle authentication in AD:

- NetNTLM -  A challenge response authentication protocol
- Kerberos - A ticket-based authentication protocol

## NetNTLM Authentication

NetNTLM (often simply called NTLM) is a challenge-response authentication protocol that has been around since the early days of Windows NT.

NTLM comes in several versions:

- NTLMv1: The original version, now considered highly insecure.
- NTLMv2: An improved version with stronger cryptography, though still vulnerable to various attacks.

Here is how NTLM authentication works:

1. The client sends a request to access a service, providing their username.
2. The server generates a random 16-byte number called a challenge (or nonce) and sends it to the client.
3. The client encrypts this challenge using the NT hash of their password and sends the response back to the server.
4. The server forwards the username, the original challenge, and the client's response to the domain controller.
5. The domain controller retrieves the user's NT hash from its database and uses it to encrypt the same challenge.
6. The domain controller compares its result with the response sent by the client. If they match, authentication is successful.
7. The server receives the result from the domain controller and grants or denies access accordingly.

The user's password is never sent over the network. It is sometimes referred to as zero-knowledge proof because the user proves they know the password without revealing it directly.

### NTLM Authentication with Impacket

Here is how you authenticate with NTLM protocol:

```bash
impacket-smbclient thm.loc/claire:'Password123!'@192.168.11.51
```

And once you are in, there will be shares. And you can use the command `shares` to reveal them. And each `share` comes with its own permission. So, you might or might not be able to access every one. But if you have permission to it, you can do that by `use SHARENAME` command. 

## Kerberos Authentication

## Weaknesses in AD Authentication

## Detections and Mitigations

## Conclusion
