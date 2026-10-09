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

Kerberos is the default authentication system in modern AD environments. It uses ticket-based system unlike NTLM and authenticats through a trusted third party: the Key Distribution Center (KDC).

The name Kerberos came from Cerberus, the three headed dog from the Greek mythology, that guards the gates of the underworld.

The difference between Kerberos and NTLM is where authentication occurs. With NTLM, you authenticate to the service you want to access, which then verifies your identity with the domain controller. With Kerberos, this is reversed; you authenticate to the domain controller first and receive tickets that you then present to services to prove your identity.

### Key Kerberos Components

- Key Distribution Center (KDC) - A service running on the domain controller that handles all ticket requests. 
- Authentication Service (AS) - The component of the KDC that verifies the user's identity and issues the initial Ticket Granting Ticket (TGT).
- Ticket Granting Service (TGS) - The component of the KDC that issues service tickets to users who present a valid TGT.
- Ticket Granting Ticket (TGT) - The initial "primary ticket" issued after successful authentication.
- Service Ticket (ST) - A ticket that grants access to a specific service. Obtained by presenting a TGT to the TGS.
- Service Principal Name (SPN) - A unique identifier for a service instance, used by Kerberos to associate a service with a specific account.
- KRBTGT Account - A special account in AD whose password hash is used to encrypt all TGTs.

### Steps of Kerberos Authentication

**Step 1**: Authentication Service Request (AS-REQ)

1. The user enters their credentials on the client machine.

2. The client sends an Authentication Service Request (AS-REQ) to the KDC, containing the username and a timestamp encrypted with the user's password hash (this is called pre-authentication).

**Step 2**: Authentication Service Response (AS-REP)

3. The KDC verifies the user's identity by decrypting the timestamp using the user's password hash stored in AD.

4. If successful, the KDC responds with an AS-REP containing:

   - A session key encrypted with the user's password hash.
   - A TGT encrypted with the KRBTGT account's password hash.

**Step 3**: Ticket Granting Service Request (TGS-REQ)

5. When the user wants to access a service, the client sends a TGS-REQ to the KDC containing:

   - The TGT received earlier.
   - The SPN of the service they want to access.
   - An authenticator (username and timestamp) encrypted with the session key.

**Step 4**: Ticket Granting Service Response (TGS-REP)

6. The KDC decrypts the TGT using the KRBTGT hash, validates the request, and responds with:

   - A Service Ticket (ST) encrypted with the target service's password hash.
   - A service session key encrypted with the original session key.

**Step 5**: Application Request (AP-REQ)

7. The client presents the Service Ticket to the target service.

8. The service decrypts the ticket using its own password hash, validates the user's identity, and grants access.

### Credential Cache (ccache) Files

On Linux systems, Kerberos stores tickets in credential cache files, commonly called ccache files. They hold the user's TGT and any service tickets they have obtained during their session. 

Things to know about ccache files:

- Default location: `/tmp/krb5cc_%{uid}`
- The `KRB5CCNAME` environment variable specifies which ccache file to use.
- The `klist` command displays tickets stored in the current ccache.
- Tools like Impacket use ccache files to authenticate without passwords.

If you can obtain a user's ccache file, you can authenticate as that user without knowing their password, an attack known as Pass-the-Ticket or Pass-the-ccache. 

### Kerberos Authentication with Impacket

First of all, you have to add domain names to the IP address, because Kerberos doesn't work with just IP addresses. So, you add the IP addresses with:

```bash
echo 192.168.11.51 SERVER1.thm.loc >> /etc/hosts 
```
You will have to use `sudo` for this, or you will have to be root. After you added the domain name, you can get the ccache file with:

```bash
impacket-getTGT thm.loc/mary:'SuperLongForKerberos123!' -dc-ip 192.168.11.100
```

That will save the `mary.ccache` in the current directory.

To use this ticket for authentication, you will have to set `KRB5CCNAME` environment variable to the ccache file.

```bash
export KRB5CCNAME=mary.ccache
```

After that, you can authenticate with:

```bash
impacket-smbclient thm.loc/mary@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

- `-k` asks to use Kerberos for authentication
- `-no-pass` indicates the usage of ticket rather than passwords

## Weaknesses in AD Authentication

## Detections and Mitigations

## Conclusion
