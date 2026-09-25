# Content Discovery

**TryHackMe Room:** [Content Discovery](https://tryhackme.com/room/contentdiscoveryx)

This write-up covers my understanding of the room and the key concepts I took away from it.

## Overview

This room covers the fundamentals of web content discovery, including manually finding common files and directories, using OSINT techniques such as Google Dorking, searching archives and repositories, enumerating S3 buckets, and using Gobuster for automated directory, subdomain, and virtual-host discovery.

When I first completed the room, Gobuster was still a relatively unfamiliar tool to me. I didn't have my own wordlists at the time either, and I hadn't yet developed the habit of selecting and managing wordlists depending on what I was trying to enumerate.

After doing many CTFs since then, Gobuster has become a normal part of my workflow. Directory enumeration with it is something I now use regularly and don't have to think much about. However, redoing the room still taught me something new: I had not realised that Gobuster could also be used for DNS/subdomain enumeration and virtual-host enumeration. In the CTFs and rooms I had encountered previously, I was much more accustomed to seeing `ffuf` or similar tools used for those tasks, while Gobuster was usually used for directory enumeration.

That made revisiting the room useful. It reminded me that becoming comfortable with a tool doesn't necessarily mean knowing everything it can do.

## Manual Discovery — Common Files

Two common files worth checking during web content discovery are:

- `/robots.txt`
- `/sitemap.xml`

They serve different purposes, but both can reveal useful paths during reconnaissance.

`robots.txt` is intended to provide instructions to web crawlers about which paths should or should not be crawled. It is not an access-control mechanism, so a path listed under `Disallow` can still be publicly accessible.

In the lab, /robots.txt contained a disallowed directory:

```web
/staff-portal
```

Visiting that path allowed me to discover one of the hidden areas of the application.

I find this one of the more interesting parts of web enumeration because it is such a simple thing to check. When you encounter a website that publishes a `robots.txt` file containing interesting paths, it can almost feel like the website has left a small map for you.

The room also demonstrated `/sitemap.xml`. In the lab, it revealed another directory:

```web
/s3cr3t-area
```
This is a good example of why content discovery does not always require complicated tools. Sometimes, simply checking files that are already exposed by the website can reveal useful information.

## Manual Discovery — Headers and Framework Stack

## HTTP Headers

One way to inspect HTTP response headers is with `curl`:

```bash
curl http://MACHINE_IP -v
```

The `-v` option enables verbose output. It displays information about the HTTP request and response, including headers and the response body.

For example, the response might contain headers such as:

```bash
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Date: Mon, 04 May 2026 10:39:13 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-FLAG: [REDACTED]
< X-FLAG: [REDACTED]
< X-Powered-By: THM-Framework
...
```

Headers can sometimes reveal useful information about the server, technologies in use, cookies, debugging information, or other data that developers did not intend to expose.

In this lab, a flag was hidden in a custom `X-FLAG` response header.

This is a useful reminder that when performing web enumeration, the page itself isn't necessarily the only source of information. The HTTP response surrounding the page can contain useful information too.

## Framework Stack

The room also demonstrated identifying the technology or framework used by a website.

In the lab scenario, there was a framework-related link at the bottom of the page. Following it led to information about a hidden directory:

```web
/thm-framework-login
```

The page also provided the credentials:

```text
admin:admin
```

Using those credentials allowed me to access the page and retrieve the flag.

From a reconnaissance perspective, identifying the underlying framework or technology stack can help determine where interesting endpoints, files, or common configuration paths might exist.

## OSINT — Search Engines & Web Tools

Content discovery doesn't always mean directly interacting with the target website. Search engines can also reveal information that has already been indexed.

One technique is Google Dorking, where search operators are used to narrow search results.

Some of the operators covered in the room were:

| Filter | Example |
| --- | --- |
| `site` | `site:tryhackme.com` |
| `inurl` | `inurl:admin` |
| `filetype` | `filetype:pdf` |
| `intitle` | `intitle:admin` |
| `intext` | `intext:password` |
| `cache` | `cache:tryhackme.com` |

If I don't remember the exact syntax for a particular search operator, Google's Advanced Search interface can also be useful for constructing the query.

## Wappalyzer 

Wappalyzer is a browser extension and web tool that can identify technologies used by websites.

Technology identification can provide useful context during reconnaissance. Knowing what framework, CMS, server software, analytics platform, or other technologies a website uses can help guide further investigation.

## OSINT — Repositories & Archives

The room also covered three places where information about a website or organisation may exist outside the current version of the website:

- Wayback Machine
- GitHub
- Amazon S3

## Wayback Machine

The Wayback Machine is an archive of websites that stores snapshots of pages over time.

I have actually used it several times since around 2020 when I needed to find information that had disappeared from a website.

I think of it almost like taking photographs of a website at different points in time. If something existed on a website in the past but has since been removed, an archived snapshot may still contain it.

This can be particularly useful during reconnaissance because an older version of a website may reveal pages, directories, files, or information that is no longer visible on the current version.

## GitHub

GitHub can also be useful during reconnaissance because repositories preserve code and, importantly, their history.

I've encountered this several times while doing CTFs. Even when something has been removed from the current version of a repository, its previous commits may still contain it.

This makes repository history worth checking when looking for accidentally exposed information or older versions of application code.

### S3 Bucket

Amazon S3 (Simple Storage Service) is a cloud storage service commonly used by organisations to store files and static website content.

A commonly encountered S3 bucket hostname format is:

```web
https://{bucket-name}.s3.amazonaws.com
```

S3 enumeration is something I haven't explored much yet, since I haven't gone deeply into cloud security or cloud-focused enumeration. However, this room introduced me to the idea that cloud storage can also form part of the attack surface during reconnaissance.

## Automated Discovery — Gobuster Fundamentals

Gobuster is a web enumeration tool written in Go. It can be used for several types of enumeration, including directory discovery, DNS/subdomain discovery, and virtual-host discovery.

During most CTFs, I have primarily used Gobuster for directory enumeration. Another tool I commonly see used for similar purposes is `ffuf`.

A basic Gobuster directory enumeration command is:

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

Here:

- `-u` — target URL
- `-w` — wordlist to use

Gobuster takes each entry from the wordlist and makes requests against the target to determine whether those paths exist.

For example, a wordlist might contain:

```web
admin
login
uploads
backup
images
```

Gobuster will then test paths such as:

```web
/admin
/login
/uploads
/backup
/images
```

## Wordlists

The wordlist is an important part of enumeration.

There are many different wordlists available, and they vary in size and purpose. A smaller wordlist can be useful when I want a quick initial scan, while larger wordlists can provide broader coverage at the cost of more requests and more time.

This was something I understood much better after gaining more CTF experience. When I first completed this room, I didn't even have my own collection of wordlists and wasn't familiar with the idea of choosing one depending on the enumeration task.

Now, selecting a suitable wordlist is just part of the process I normally consider before running Gobuster.

## Automated Discovery — Subdomains & Virtual Hosts

When performing web enumeration, it is important not to restrict discovery to the main domain.

For examples: 

```web
example.thm
mobile.example.thm
```

`mobile.example.thm` is a subdomain of `example.thm`.

A separate subdomain may point to a different server or application, and it may have a different configuration or attack surface from the main website.

## Subdomain vs virtual Hosts

This was one of the concepts from the room that I had not fully understood before revisiting it.

A subdomain is a hostname within a domain hierarchy. It can be resolved through DNS to an IP address.

For example:

```bash
example.thm          → 10.10.10.50
mobile.example.thm   → 10.10.10.50
api.example.thm      → 10.10.10.51
```

The important point is that each subdomain does not necessarily need its own unique IP address. Multiple hostnames can resolve to the same IP address.

A virtual host, on the other hand, is a web-server configuration that allows multiple websites or applications to be hosted on the same IP address. The server can use the hostname supplied in the HTTP Host header to determine which site should handle the request.

For example, these could all point to the same IP:

```bash
example.thm          → 10.10.10.50
mobile.example.thm   → 10.10.10.50
api.example.thm      → 10.10.10.50
```

The web server can then distinguish between them based on the requested hostname:

```bash
Host: blog.example.thm
```

Conceptually:

```text
Subdomain
    ↓
DNS resolution
    ↓
IP address

Virtual Host
    ↓
HTTP Host header
    ↓
Web server configuration
    ↓
Which website should handle the request?
```

This distinction is important during enumeration because finding a hostname through DNS and finding a virtual host on a web server are related, but they are not the same thing.

## Gobuster DNS Mode

Gobuster can perform DNS enumeration using its `dns` mode:

```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```

Flags:

- `-d` / `--domain` — target domain
- `-i` / `--show-ips` — show the IP addresses discovered
- `-r` / `--resolver` — use a custom DNS resolver

The wordlist contains potential subdomain names, which Gobuster attempts to resolve.

For example, if the wordlist contains:

```web
admin
api
blog
dev
mobile
```

Gobuster can test names such as:

```web
admin.example.thm
api.example.thm
blog.example.thm
dev.example.thm
mobile.example.thm
```

## Gobuster Virtual Host Mode

Gobuster can also enumerate virtual hosts using `vhost` mode:

```bash
gobuster vhost -u "http://MACHINE_IP" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

Here:

- `--append-domain` — appends the target domain to each word in the wordlist
- `--exclude-length` — excludes responses of specified lengths, which can help filter out false positives

The important difference from DNS enumeration is what is being tested.

With DNS enumeration, Gobuster is checking whether potential hostnames can be resolved through DNS.

With virtual-host enumeration, Gobuster is making web requests with different hostnames and looking at how the web server responds.

## `/etc/hosts` and `DNS` Configuration

Depending on the environment, some of the setup required for these exercises involves configuring local hostname resolution or DNS resolution.

For example, `/etc/hosts` can be used to create a local hostname-to-IP mapping:

```bash
10.10.10.50    example.thm
```

This tells my machine to associate `example.thm` with `10.10.10.50` locally.

This is different from configuring a DNS resolver. `/etc/hosts` provides local mappings, while DNS resolver configuration determines where the system sends DNS queries when it needs to resolve a hostname.

Understanding this distinction became particularly useful while working through the subdomain and virtual-host enumeration sections.

## Conclusion

What made this room particularly interesting to me was doing it twice.

When I first completed it one or two months ago, Gobuster was still a tool I was getting familiar with. I didn't have my own wordlists, and I wasn't yet comfortable enough with web enumeration for many of these concepts to feel natural.

After doing many CTFs, that has changed. Gobuster directory enumeration has become a normal part of my workflow. I now think about wordlists, enumeration scope, and the type of discovery I want to perform much more naturally.

At the same time, redoing the room showed me that becoming comfortable with a tool doesn't mean I know everything about it. I had never really considered using Gobuster for DNS/subdomain or virtual-host enumeration because most of the examples I had encountered used tools such as `ffuf` for those tasks.

I also enjoyed the manual discovery side of the room. Something as simple as checking `robots.txt` can sometimes reveal an interesting path, and I find it particularly satisfying when I encounter a real website that has useful information sitting there in plain sight.

For me, the biggest takeaway from revisiting this room wasn't a single command or tool. It was seeing the difference between knowing something because I read about it once and knowing it because I have encountered it repeatedly in practice. Repeating an older room after gaining more hands-on experience can reveal things that I completely missed the first time.
