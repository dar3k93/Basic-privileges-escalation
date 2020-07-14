#### SUID/SGID
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

#### Use strace for track suid file execute
```
strace /suid/file/path 2>&1 | grep -iE "open|access|no such file"
strace -v -f -e execve /suid/file/path 2>&1 | grep service
```
