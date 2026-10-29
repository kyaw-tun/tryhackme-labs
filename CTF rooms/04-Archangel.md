# Archangel

> **TryHackMe Room**: [Archangel](https://tryhackme.com/room/archangel)

This write-up covers how I approached the room, step by step. Flags are not included.

## Initial Access

The Nmap scan showed ports `22` and `80` open. The web page on port `80` gave the first answer directly, and the second required adding the machine’s hostname to my `/etc/hosts` file to reach it by name.

I ran `gobuster` for directory enumeration but found nothing useful at first. It turned out I was scanning the wrong hostname: the intended virtual host served a different, in-development site than the default one on the same IP. The same IP address can serve different sites depending on the hostname in the request (virtual hosting), so scanning the default host missed the development site entirely. Once I enumerated against the correct hostname, the development site and its pages appeared.

On the development site I found a `test.php` page with a `view` parameter, a likely LFI. My first attempt with a normal traversal payload returned:

```text
Sorry, Thats not allowed
```

So the input was being filtered. To understand how, I read the page’s own source. Because including a `.php` file directly would execute it rather than show it, I used the `php://filter` wrapper to base64-encode the file first, which returns the raw source as text:

```url
php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

Decoding the base64 gave me the source:

```php
<?php
function containsStr($str, $substr) {
    return strpos($str, $substr) !== false;
}
if(isset($_GET["view"])){
    if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
        include $_GET['view'];
    } else {
        echo 'Sorry, Thats not allowed';
    }
}
?>
```

The rule is: the path must not contain `../..`, and it must contain `/var/www/html/development_testing`. So I needed a traversal that reaches other files while avoiding the literal string `../..` and still including the allowed directory. Variations such as `.././../` and `..//..//` get around the `../..` check while still resolving to a traversal on the filesystem.

### From LFI to a shell (log poisoning)

Since test.php uses include rather than readfile, this LFI can be pushed from reading files to executing code, through log poisoning. The idea: the Apache access log records the User-Agent of each request, so if I send a request whose User-Agent contains PHP code, that code is written into the log. Then, by including the log file through the LFI, the code gets executed.

Working in Burp Suite, I set the User-Agent to a small PHP payload that runs a command from a parameter:

```php
<?php system($_GET['cmd']); ?>
```

Then I included the access log through the LFI (as a plain include, not through `php://filter`, since the point now is to execute the planted code, not read it), reaching it with a traversal that satisfied the filter:

```url
/var/www/html/development_testing/..//..//..//log/apache2/access.log
```

With the payload in place, adding `&cmd=` to the URL ran commands as `www-data`. It sometimes took a couple of refreshes to take effect.

From there I used a reverse shell to get an interactive session. I tried more than one form of the reverse-shell command; the one that worked wrapped it in `bash -c '...'`. [To confirm: the exact reason the other form failed — I will verify this.] I caught the shell with a `netcat` listener and landed as `www-data`, then read the flag.

## Privilege escalation to archangel (cron + writable script)

Looking for a path onward, I checked `/etc/crontab` and found a job running every minute as the archangel user, executing `/opt/helloworld.sh`:

```bash
drwxrwxrwx  3 root      root      ... .
-rwxrwxrwx  1 archangel archangel  66 ... helloworld.sh
```

The script was world-writable (`rwxrwxrwx`), so I could replace its contents with a reverse shell. With a `netcat` listener ready, the cron job ran my code on its next minute and connected back as `archangel`. I read the next flag.

## Privilege escalation to root (PATH hijacking)

In archangel’s files I found a SUID binary (with another flag alongside it). Inspecting what it did, it copied files using `cp` without a full path:

```bash
cp /home/user/archangel/myfiles/* /opt/backupfiles
```

Because it calls `cp` by name rather than `/bin/cp`, the system searches the directories in `$PATH` to find `cp`. If I put a directory I control at the front of `$PATH`, I can supply my own `cp`. So I created a fake `cp` in a writable directory containing a shell:

```bash
#!/bin/bash
/bin/bash
```

I made it executable and put its directory first in `$PATH`:

```bash
export PATH=$PWD:$PATH
```
When the SUID binary ran, it executed my `cp` instead of the real one, with the binary’s elevated privileges, giving me a root shell:

```bash
uid=0(root) ...
```

And I read the final flag.

## What I took away

Archangel is a clean example of chaining several common techniques: a filtered LFI read with `php://filter`, turning that LFI into code execution through log poisoning, a world-writable cron script for the first privilege escalation, and a SUID binary calling a command without a full path for the second (PATH hijacking). The recurring lesson is that each step depended on reading carefully, the exact filter logic, the file permissions, how a binary calls its helpers, rather than on any single payload.
