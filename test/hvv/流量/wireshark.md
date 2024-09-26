### Wireshark过滤

```
逻辑操作：and or not(相当于!)
eq等同于==

ip.addr == 192.168.1.5
源ip：ip.src eq 192.168.1.5
目标ip：ip.dst eq 192.168.1.107
端口（源和目的一个符合即可）：tcp.port eq 80 (也可为udp协议)
只显tcp协议的目标端口80：tcp.dstport == 80 
只显tcp协议的来源端口80：tcp.srcport == 80 

直接输入协议即可过滤：tcp，udp，arp，icmp，http，smtp，ftp，dns，msnms，ip，ssl，oicq，bootp

过滤目标mac：eth.dst == A0:00:00:04:C5:84 
过滤来源mac：eth.src eq A0:00:00:04:C5:84 

http请求方式过滤：http.request.method == "GET"
http内容过滤:http contains "HTTP/1.1 200 OK"
```
