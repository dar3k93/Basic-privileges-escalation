### Proxytunnel

#### use squid proxy
- proxytunnel -p 192.168.0.115:3128 -d 127.0.0.1:22 -a 1234
#### e.g:
- ssh name@127.0.0.1 -p 1234

### Proxychains 
#### use squid proxy
- nano /etc/proxychains.conf
#### add:
- http 192.168.213.141 3128

#### scan ports via tunnel
- proxychain nmap -sT -p 22 127.0.0.1

#### ssh login via proxychains
- proxychains ssh name@127.0.0.1

### SSH port forwarding
In case exploit can not be run locally on target machine, you can frowarded the port via SSH to your local machine
```
ssh -R [local_port]:127.0.0.1:[service_port] [username]@[your_kali_ip]
```
Exploit code can be run on local machine

#### case study
##### on victim machine
- check service port *netstat -nl*
- ssh -R 4444:127.0.0.1:3306 root@192.168.100.104

##### on yours kali machine
mysql -u root -h 127.0.0.1 -P 4444
