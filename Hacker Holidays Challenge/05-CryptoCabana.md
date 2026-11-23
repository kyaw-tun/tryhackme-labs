# CryptoCabana

> **TryHackMe Room:** [CryptoCabana](https://tryhackme.com/room/hh-cryptocabana-f81cac95)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This is my second cloud room after [Complimentary](02-Complimentary.md). It is rated medium in TryHackMe's list of Hacker Holidays Challenge rooms, with a duration of 60 minutes.

There are two ways to complete this lab: through the Azure portal and its built-in terminal, or by installing the Azure CLI on one's own machine. I tried installing it locally and succeeded, but ultimately completed the lab through the Azure portal instead.

## Getting the Flag

The lab provided a link to a website. Reviewing its source code and `js` file revealed hardcoded credentials. Here is the target link:

```URL
https://cryptocabanaf5scjagc.z13.web.core.windows.net/
```

The hardcoded credentials were a storage account name and a SAS token. In the Azure Linux terminal, I set both to variables for easy reuse.

With these, I had access equivalent to a low-level service account. I first listed the containers in the storage account:

```bash
az storage container list --account-name <ACCOUNT> --sas-token <SAS>
```

This returns the list in JSON format, which becomes difficult to read once there are multiple entries. I simplified it by requesting table format instead:

```bash
az storage container list --account-name <ACCOUNT> --sas-token <SAS> -o table
```

This showed three containers. One was empty, and one was inaccessible, leaving only one to explore. I checked its contents:

```bash
az storage blob list --account-name <ACCOUNT> --container-name <CONTAINER> --sas-token <SAS> -o table
```

It contained two files, one `txt` and one `json`. I downloaded both for easier access:

```bash
az storage blob download --account-name <ACCOUNT> --container-name <CONTAINER> --name <NAME-1> --sas-token <SAS> --file <DOWNLOAD-1.txt> -o none
az storage blob download --account-name <ACCOUNT> --container-name <CONTAINER> --name <NAME-2> --sas-token <SAS> --file <DOWNLOAD-2.json> -o none
```

The `txt` file contained a list of words typically used to memorize multi-factor authentication codes. The `json` file was written on a single line, making it hard to read. I reformatted it with:

```bash
jq . <DOWNLOAD-2.json>
```

This revealed several hardcoded, sensitive values belonging to a client: a client ID, client secret, tenant ID, and vault name. I set the client ID, secret, and tenant ID as variables, but overlooked the vault name until I needed it a step later. With these, I could log in to Azure as that client:

```bash
az login --service-principal --username <CLIENT_ID> --password <CLIENT_PASSWORD> --tenant <TENANT_ID> --allow-no-subscriptions -o none
```

On my first attempt, this returned an error, since I had not yet set the client ID and secret correctly beforehand. Once I corrected them and confirmed I was logged in, I checked the contents of the vault:

```bash
az keyvault secret list --vault-name <VAULT_NAME> --o table
```

This listed three keys. I accessed each one with:

```bash
az keyvault secret show --vault-name <VAULT_NAME> --name <KEY1> --o table
az keyvault secret show --vault-name <VAULT_NAME> --name <KEY2> --o table
az keyvault secret show --vault-name <VAULT_NAME> --name <KEY3> --o table
```

These keys together form the flag. `Key1` and `Key3` were accessible directly, but `Key2` returned a message stating that it had been rotated. The room provided a clue pointing toward this:

```clue
"the backup kiosk is SO confident. 'sleep easy' it says 💀 reader, do not sleep easy. also: if a value looks freshly rotated, ask yourself what it looked like five minutes before that 👀"
```

Following the hint, I listed all versions of `Key2`:

```bash
az keyvault secret list-versions --vault-name <VAULT_NAME> --name <KEY2>
```

Accessing a version prior to the current one revealed the missing part of the flag.

## Conclusion

The underlying concept of this room is fairly simple. Starting from a storage account and its token, I looked for containers I could access, found one, and discovered hardcoded credentials and a vault name inside it. Those credentials granted access to the vault, which held the final pieces of the flag. Working through it, though, made clear how little I actually knew about cloud environments at the time. I made basic mistakes, like forgetting to set a variable before using it, and it took real effort to get through a room whose underlying logic was not complicated at all.

I could follow and explain the logic behind every step, but I relied on outside explanations both times I went through this room, and I do not want to present it as something I solved unaided. Cloud security is clearly an area I need more independent practice in before I would call myself comfortable with it.
