#### 信息收集

1. 主机发现：nmap -sn -PE -n 192.168.1.0/24
   192.168.1.207是新增加的IP地址
2. 端口扫描：nmap -sT --min-rate 10000 -p- 192.168.1.207
   PORT      STATE SERVICE
   80/tcp    open  http
   139/tcp   open  netbios-ssn
   445/tcp   open  microsoft-ds
   10000/tcp open  snet-sensor-mgmt
   20000/tcp open  dnp
3. 获取详细服务信息以及操作系统信息：nmap -sT -sV -O --min-rate 10000 -p80,139,445,10000,20000 192.168.1.207

   PORT      STATE SERVICE     VERSION
   80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
   139/tcp   open  netbios-ssn Samba smbd 4.6.2
   445/tcp   open  netbios-ssn Samba smbd 4.6.2
   10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
   20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
4. 进行gobuster目录扫描：gobuster dir -u http://192.168.1.207 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

   同时访问80端口

#### Web渗透

1. 80端口为Apache服务的默认页面，但查看页面源代码，在最后一行发现了brainfuck代码：

   ```
   <!--
   don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

   ++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.


   -->
   ```

   解密后得到：`.2uqPEfj3D<P'a-3`
2. 了解SMB服务：

   ```
   SMB（服务器消息块）是一种协议，它允许同一网络上的资源共享文件，浏览网络并通过网络进行打印。
   ```

   利用工具Enum4linux获取信息：Enum4linux 192.168.1.207
   信息检索后：S-1-22-1-1000 Unix User\cyber (Local User)
   得到用户名
3. 登录Webmin，10000端口失败，20000端口成功登录
   进入网页自带终端，拿到初级用户权限

3mp!r3{You_Manage_To_Break_To_My_Secure_Access}

#### 权限提升

1. 发现当前目录下有一个tar可执行文件，ls -l，发现所有用户都有可执行权限，可以利用tar将想要读取的文件压缩，再以普通用户解压缩，可以读取任意文件
2. 查找敏感文件，在var目录下发现了backups目录，进入backups目录，发现.old_pass.bak隐藏文件
3. 将其压缩到家目录下，在解压缩，查看文件内容：

   Ts&4&YurgtRX(=~h
4. su root，成功切换到root用户

```
3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}

Author: Icex64 & Empire Cybersecurity
```
