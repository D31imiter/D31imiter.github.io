#### 用户信息

1. 查看三个账户文件，/etc/passwd，/etc/sudoers，/etc/shadow文件
   * stat查看修改时间
   * 是否有新增内容：/etc/passwd中是否新增了可以以/bin/bash（或其他shell）登陆的用户，是否新增了特权用户（uid为0），/etc/sudoers中是否新增了ALL=(ALL) ALL字样，/etc/shadow是否有新增
2. 查看当前登录用户，以及其登录ip。pts代表远程登录，tty代表本地登陆：`who`
3. 查看目前登入系统的用户，以及他们正在执行的程序：`w`
4. 查看现在的时间、系统开机时长、目前多少用户登录，系统在过去的1分钟、5分钟和15分钟内的平均负载：`updtime`

#### 查看历史命令

```
history		命令查看当前用户历史命令
.bash_history	查看各用户家目录下的历史命令信息
```

#### 查看开放端口

```
netstat
-a (all)显示所有选项，默认不显示 LISTEN 相关
-t (tcp)显示tcp相关选项
-n 不显示别名，能显示数字的全部转化成数字
-p 显示建立相关链接（sockets）的程序名
```

如果发现可疑外联IP，即可根据对应PID查找其文件路径

#### 查看进程

```
ps:
详见Linux
```

查看cpu占用率前十的进程：`ps aux --sort=pcpu | head -10`

#### 查看自启项

`systemctl list-unit-files | grep enabled`

#### 查看定时任务

`crontab  -l`

查看指定用户定时任务：`crontab -u [user] -l`

#### 进程监控

```
ps	列出进程
top	按cpu占用率排序，输入b按内存占用排序
```

#### 登录日志

`last -f /var/log/wtmp`

该日志文件永久记录每个用户登录、注销及系统的启动、停机的事件

`/var/log/secure` 

查看可疑IP登录次数

#### 命令状态

查看命名修改时间，防止被替换：`stat /bin/netstat`
