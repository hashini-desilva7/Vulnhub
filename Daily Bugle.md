nmap -sn 192.168.56.0/24

ssh
http
mysql

searchsploit OpenSSH 7.4 - look if this is exploitable
searchsploit httpd 2.4.6 - look if this can be exploited
searchsploit PHP/5.6.40 - look if this can be exploited

paste the ip in the browser
1. there's a photo in the website- download it and  analyse (if it is actually a png/jpg,if it has hidden data)
if i see any tab i can click on those and see - if it is sql injection

2. go to source code- nothing found
3. look for sub directories

```
dirb <url>/ --wordlist 
```

if i need my own wordlist i can use that option
and there's /robot.txt , /administrator.txt , /media, README.txt (found the joomlah version from here)

go to robot.txt

searchsploit Joomla 3.7.0

look for a exploit in GTFobins

python3 joomblah.py ip>
and we found a hash and decoded it from hashes.com and found the password ==bcrypt==

touch hash
nano hash
paste the hash and save 
john hash --format=bcrypt  hash
