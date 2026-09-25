# Content Discovery

This room contains basic web knowledge: such as manually discovering web hidden contents like, robots.txt, sitemap.xml, using OSINT tools such as google dorking, using wayback machine, GitHub, and S3 bucket enumeration, using gobuster to brute force directories, and applying content discovery methodology in a penetration test.

## Manual Discovery - Common Files

There are two directories that most sites have, `/robots.txt` and `/sitemap.xml`. Many websites mostly have `/sitemap.xml` rather than `/robots.txt`. But one should check for both at any time. And in this room, on the lab, there is a **disallow** directory listed on `/robots.txt` which is `/staff-portal`. And then you can find `/s3cr3t-area` directory in the `/sitemap.xml` file.

## Manual Discovery - Headers and Framework Stack

### HTTP Headers

You can use `curl` for this. 

```bash
curl http://MACHINE_IP -v
```

- `-v` means verbose output

What it does it, gives out the whole web content in HTML code from the Headers. And there, you can even find the hidden flags some developers might hide. Here in this lab, there was a flag hidden in the `X-FLAG` section. 

### Framework Stack

And here in this lab scenario, there was a framework site link at the bottom of the page. And you can find hidden directory `/thm-framework-login` and it is written that you can login using `admin` and `admin` as credentials. And you can find the flag there. 

## OSINT - Search Engines & Web Tools

There are multiple ways to google dorking, such as adding distinct commands to narrow down search results, and if you can't remember the exact command, you can try Advanced Google Search in the Google site. Here are some commands taught in the room:

| Filter | Example |
| --- | --- |
| `site` | `site:tryhackme.com` |
| `inurl` | `inurl:admin` |
| `filetype` | `filetype:pdf` |
| `intitle` | `intitle:admin` |
| `intext` | `intext:password` |
| `cache` | `cache:tryhackme.com` |

### Wappalyzer 

It is a browser extension and web tool that can identify technologies that websites use. 

## OSINT - Repositories & Archives

There are three tools you can use as archives of some sorts. Wayback Machine, GitHub, and S3 buckets.

### Wayback Machine

This is like an ultimate website archives. You can even find the content that has been deleted on the internet. I have tried this multiple times since 2020 whenever I need it. It kind of acts like an image. For example, it takes "picture" of a website multiple times a year and it archives it there on their server. And then later in the future, you can access it like a time travel machine, hence the name **wayback machine**.

### GitHub

GitHub acts in a very similar way to the wayback machine. But it captures the code and stores it in the history. I have used this feature in many CTF rooms, but not as a personal need. 

### S3 Bucket

As per TryHackMe, Amazon S3 (Simple Storage Service) is a cloud storage platform that many organisations use to host files and static website content.
The URL format for an S3 bucket is `https://{name}.s3.amazonaws.com`. I haven't used it yet as I have yet to explore Cloud security and AI security.

## Automated Discovery - Gobuster Fundamentals

Gobuster is a web directory enumerating tool written in Go language. During most CTFs, you have to use it or other web enumerating tool like `ffuf`.

There are multiple ways to enumerate web directories. The most common/easy one is running: 

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

The path to wordlist may differ as I have my own on `/usr/share/wordlists/dirbuster/` directory. What that command does is enumerate the web Url based on the most common directory names. So, it will try to enter each word from that list and check if it exist.

Here:

- `-u` - target URL
- `-w` - target wordlist

Wordlist is important when using GoBuster as there are multiple common ones. You will have to choose based on your own needs, as some provided expansive wordlists but they take too much time to finish enumerating. And not all fast ones will cover all of the wordlist that the website might contain.

## Automated Discovery - Subdomains & Virtual Hosts

Gobuster is a web directory enumerating tool written in Go language. During most CTFs, you have to use it or other web enumerating tool like `ffuf`.

There are multiple ways to enumerate web directories. The most common/easy one is running: 

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

The path to wordlist may differ as I have my own on `/usr/share/wordlists/dirbuster/` directory. What that command does is enumerate the web Url based on the most common directory names. So, it will try to enter each word from that list and check if it exist.

Here:

- `-u` - target URL
- `-w` - target wordlist

Wordlist is important when using GoBuster as there are multiple common ones. You will have to choose based on your own needs, as some provided expansive wordlists but they take too much time to finish enumerating. And not all fast ones will cover all of the wordlist that the website might contain.

## Automated Discovery - Subdomains & Virtual Hosts

Examples: 

- Normal domain - `tryhackme.thm`
- subdomain - `mobile.tryhackme.thm`

When you are doing web enumeration, you should always enumerate for subdomains too. For example, if there was a vulnerability in the website, they might have fixed it for the main website (`tryhackme.thm`) but not for the subdomain (`mobile.tryhackme.thm`).

### Subdomain vs virtual Hosts

- Subdomain - It is resolved through DNS. You have to assign ip addresses to each subdomain. 
- Virtual Hosts - It is resolved through web server. Meaning, one ip address can run multiple websites, with the server using the `Host:` HTTP Header

And when enumerating subdomain and virtual hosts in `gobuster`, you can use `dns` mode and `vhost` mode. The rooms had shown some examples. But first, you have to update some files, in order to do that. Like adding a ip address and domain/subdomain to `/etc/hosts` file and adding ip address to `/etc/resolv-dnsmasq` file. 

### DNS mode command example

```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```

- `-d` - domain (long form is `--domain`) 

Other flags:

- `-i` - show ip (long form is `--show-ips`)
- `-r` - use a custom dns for lookup (long form is `--resolver`)

### vhost mode command example

```bash
gobuster vhost -u "http://MACHINE_IP" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```
- `--append-domain` - combine each wordlist with the domain
- `--exclude-length` - filters out false positives

## Conclusion
