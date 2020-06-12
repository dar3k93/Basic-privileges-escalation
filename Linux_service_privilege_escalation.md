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

```Example
myqld --version
```
