#### windows计划任务

通过输入 `taskschd.msc`打开计划任务，也可以通过 `schtasks`命令来管理计划任务

`schtasks`将命令和程序计划为定期运行或在特定时间运行、在计划中添加和删除任务、按需启动和停止任务，以及显示和更改计划任务。

```
schtasks /change		更改任务的以下一个或多个属性：
					任务运行的程序 (/tr)
					运行任务的用户帐户 (/ru)
					用户帐户的密码 (/rp)
					将仅交互属性添加到任务 (/it)
schtasks /create		计划新任务
schtasks /delete		删除计划任务
schtasks /end			停止任务启动的程序
schtasks /query			显示计划在计算机上运行的任务。
schtasks /run			立即启动计划任务。 运行操作会忽略计划，而是使用任务中保存的程序文件位置、用户帐户和密码立即运行任务
```

**schtasks /Create**

```
SCHTASKS /Create [/S system [/U username [/P [password]]]]
    [/RU username [/RP password]] /SC schedule [/MO modifier] [/D day]
    [/M months] [/I idletime] /TN taskname /TR taskrun [/ST starttime]
    [/RI interval] [ {/ET endtime | /DU duration} [/K] [/XML xmlfile] [/V1]]
    [/SD startdate] [/ED enddate] [/IT | /NP] [/Z] [/F]
```

schtasks用于创建计划任务，在Windows Server 2008之前使用at命令创建计划任务，Windows Server 2008之后才有schtasks命令

1. /s ：指定主机
2. /tn ：指定
3. /sc : 指定计划类型，详见下

   ```
   MINUTE、HOURLY、DAILY、WEEKLY、MONTHLY	指定计划的时间单位。

   ONCE	任务在指定的日期和时间运行一次。
   ONSTART	任务在每次系统启动的时候运行。可以指定启动的日期，或下一次系统启动的时候运行任务。
   ONLOGON	每当用户（任意用户）登录的时候，任务就运行。可以指定日期，或在下次用户登录的时候运行任务。
   ONIDLE	只要系统空闲了指定的时间，任务就运行。可以指定日期，或在下次系统空闲的时候运行任务。
   ```
4. /mo : 指定任务在其计划类型内的运行频率。这个参数对于 MONTHLY 计划是必需的。对于 MINUTE、HOURLY、DAILY 或 WEEKLY 计划，这个参数有效，但也可选。默认值为 1。

   ```
   MINUTE	1 ～ 1439	任务每 n 分钟运行一次。
   HOURLY	1 ～ 23		任务每 n 小时运行一次。
   DAILY	1 ～ 365	任务每 n 天运行一次。
   WEEKLY	1 ～ 52		任务每 n 周运行一次。
   MONTHLY	1 ～ 12		任务每 n 月运行一次。
   LASTDAY	任务在月份的最后一天运行。
   FIRST、SECOND、THIRD、FOURTH、LAST	与 /d day 参数共同使用,并在特定的周和天运行任务。例如，在月份的第三个周三。
   ```
5. /tr : 指定任务运行的程序或命令
6. /ru : 使用指定用户帐户的权限运行任务
7. /f : 强制执行，阻止确认消息
8. /u指定用户，/p指定密码

#### net

windows中的网络服务都使用以net开头的命令，使用它可以轻松的管理本地或者远程计算机的网络环境，以及各种服务程序的运行和配置，或者进行用户管理和登陆管理等。

1. 共享管理
   * 查看当前主机的共享 `net share`
   * 查看指定计算机的共享资源列表 `net view \\[hostname]`
   * 查看指定域的共享资源列表 `net view /domain:[domainname]`（需要关闭防火墙）
   * 添加共享 `net share [sharename]=[local path] `
2. 用户管理
   * 添加用户：

     ```
     添加用户设置明文密码：net user /add [username] [password]
     设置密码时不可见：net user /add [username] *

     用户名后加$表示隐藏账户，不会再net user列表中出现
     ```
   * 删除用户：`net user /del [username]`
   * 设置某个用户的状态为启用(禁用)：`net user /active:yes(no) [username]`
   * 组管理（用于域管理，本地组管理使用 `localgroup`参数）：

     ```
     查看所有组：net group
     查看组中的用户：net group [groupname]
     将用户添加到组：net group [groupname] /add [username list]
     将用户从组删除：net group [groupname] /del [username list]
     添加组注释：net group [groupname] /comment:[text]
     ```
3. 连接及资源管理
   * 查看共享连接 `net use`
   * 建立共享连接 `net use \\[ip]\ipc$ [password] /user:[domainname]\[username]`
   * 创建映射 `net use [local file] [target file]`

#### SC

SC 是用来与服务控制管理器和服务进行通信的命令行程序

`sc [servername] create [name] binpath="[path]"`
