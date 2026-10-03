apache default location= /vat/www/html/
then find bash based shells instead of php shell5

ls |grep .txt and found credentials.txt


edit ip and port number 
enter in the ggl
enter in victim machine she;; asw
su armour and the md5 hash we got before as the pwd

DFIR Training- analyse RAM
digital cortpora 



first i identified the ip in the victim machine and added it to the /etc/hosts

```
kali㉿hashi)-[~]
└─$ sudo nano /etc/hosts
```

then i scanned the ip
`nmap -p- escalate-my-priviledge
`
and i found http ,ssh as the open ports 
```
PORT      STATE  SERVICE
22/tcp    open   ssh
80/tcp    open   http
111/tcp   open   rpcbind
875/tcp   closed unknown
2049/tcp  open   nfs
20048/tcp open   mountd
42955/tcp closed unknown
46666/tcp closed unknown
54302/tcp closed unknown
```

then i navigated to http ip and tried to click the image and although i was directed to a diff website i couldnt find anything useful there then i checked th source code and found a php bash shell

![[Pasted image 20260906214018.png]]

then i tried to log into it 
`http://192.168.56.118/phpbash.php```

and i got a shell there 
since that was not a interactive shell to get a interactive shell 
first i started a listener in my kali machine
`nc -lnvp 1234 `

then i found a bash reverse shell from pentest monkey edited the ip and the port of it and Then, on the webpage command execution input, I ran the command

then i go


