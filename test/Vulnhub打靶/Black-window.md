#### 信息收集

1. 主机发现：`nmap -sn -PE -n 192.168.1.0/24`
   192.168.1.118是新增加的IP，是靶机
2. 端口扫描：`nmap -sT --min-rate 10000 -p- 192.168.1.118`
   ```
   PORT      STATE SERVICE
   22/tcp    open  ssh
   80/tcp    open  http
   111/tcp   open  rpcbind
   2049/tcp  open  nfs
   3128/tcp  open  squid-http
   38519/tcp open  unknown
   40003/tcp open  unknown
   43033/tcp open  unknown
   45293/tcp open  unknown
   ```
3. 获取详细信息以及操作系统版本：`nmap -sT -sV -O -A -p22,80,111,2049,3128,38519,40003,43033,45293 192.168.1.118`
   ```
   PORT      STATE SERVICE     VERSION
   22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
   | ssh-hostkey: 
   |   2048 f83b7ccac2f65aa60e3ff9cf1ba9dd1e (RSA)
   |   256 04315a34d49b1471a00f22782df3b6f6 (ECDSA)
   |_  256 4e428e69b790e82768df688a83a7879c (ED25519)
   80/tcp    open  http        Apache httpd 2.4.38 ((Debian))
   |_http-server-header: Apache/2.4.38 (Debian)
   |_http-title: Site doesn't have a title (text/html).
   111/tcp   open  rpcbind     2-4 (RPC #100000)
   | rpcinfo: 
   |   program version    port/proto  service
   |   100000  2,3,4        111/tcp   rpcbind
   |   100000  2,3,4        111/udp   rpcbind
   |   100000  3,4          111/tcp6  rpcbind
   |   100000  3,4          111/udp6  rpcbind
   |   100003  3           2049/udp   nfs
   |   100003  3           2049/udp6  nfs
   |   100003  3,4         2049/tcp   nfs
   |   100003  3,4         2049/tcp6  nfs
   |   100005  1,2,3      45293/tcp   mountd
   |   100005  1,2,3      45549/udp   mountd
   |   100005  1,2,3      51127/tcp6  mountd
   |   100005  1,2,3      54016/udp6  mountd
   |   100021  1,3,4      34497/udp6  nlockmgr
   |   100021  1,3,4      40099/tcp6  nlockmgr
   |   100021  1,3,4      43033/tcp   nlockmgr
   |   100021  1,3,4      49722/udp   nlockmgr
   |   100227  3           2049/tcp   nfs_acl
   |   100227  3           2049/tcp6  nfs_acl
   |   100227  3           2049/udp   nfs_acl
   |_  100227  3           2049/udp6  nfs_acl
   2049/tcp  open  nfs_acl     3 (RPC #100227)
   3128/tcp  open  squid-http?
   38519/tcp open  mountd      1-3 (RPC #100005)
   40003/tcp open  mountd      1-3 (RPC #100005)
   43033/tcp open  nlockmgr    1-4 (RPC #100021)
   45293/tcp open  mountd      1-3 (RPC #100005)
   OS details: Linux 4.15 - 5.6
   ```
4. 漏洞扫描：`nmap --script=vuln -p22,80,111,2049,3128,38519,40003,43033,45293 192.168.1.118`
   有Dos，xss，Csrf等漏洞，但基本无法利用，扫除了三个比较有价值的目录
   ```
   p-enum: 
   |   /company/: Potentially interesting folder
   |   /docs/: Potentially interesting directory w/ listing on 'apache/2.4.38 (debian)'
   |_  /js/: Potentially interesting directory w/ listing on 'apache/2.4.38 (debian)'
   ```

#### Web渗透

1. 应该是php和Apache服务，目录爆破 `gobuster dir -u http://192.168.1.118 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html`，查看80端口，是一张黑寡妇蜘蛛的图片，下载查看是否有隐写，无可以利用的信息
2. 查看当前得到的敏感目录/doc，/js都是空目录，查看/company目录，是一个由Bootstrap搭建的网站，查看网页源代码，发现这样一句话：
   `We are working to develop a php inclusion method using "file" parameter - Black Widow DevOps Team.`，利用file参数进行文件包含，测试同时再次开启目录爆破，爆破出started.php以及一个文件列表，文件列表大多是css和js文件，无可以利用的信息，查看started.php

   1. get提交file参数页面为空白，提交其他参数或不提交页面正常
   2. html页面不管有无参数都是原页面

   file参数很可疑，尝试包含../../../../../../../../../../../../../etc/passwd恰好包含passwd文件，../../../../../../../../../../../../../为根目录
3. 读取日志文件，`file=../../../../../../../../../../../../../var/log/apache2/access.log`，无法读取，网上查了之后才知道是目录爆破次数太多了，导致文件太大无法读取，上一台靶机Red也是这样乱扫，导致磁盘空间满了下载不下来后门文件，谨慎爆破，进入安全模式后清空日志文件，被设置了root密码，删不了，G，下一台靶机
