### Proxytunnel
#### use squid proxy
- proxytunnel -p 192.168.0.115:3128 -d 127.0.0.1:22 -a 1234
#### e.g:
- ssh name@127.0.0.1 -p 1234

### Proxychains 
- nano /etc/proxychains.conf
#### add:
- http 192.168.213.141 3128

#### scan ports via tunnel
- proxychain nmap -sT -p 22 127.0.0.1

#### ssh login via proxychains
- proxychains ssh name@127.0.0.1
