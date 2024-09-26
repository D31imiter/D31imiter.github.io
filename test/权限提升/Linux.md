### Linux权限提升

#### 信息收集

上传信息收集脚本到/tmp目录下执行，例如linux-smart-enumeration，LinEnum，也可以手动信息收集，对提权有用的的信息：

1. 查找有suid权限且属主为root的二进制文件:
   find / -user root -perm -u=s 2>/dev/null
2. 查看当前用户有哪些可执行的命令，sudo -l

#### 内核漏洞提权

* uname -r命令查看内核版本，searchsploit linux kernel [内核版本] | grep 'Privilege Escalation'，利用kali漏洞库exploit DB搜索有具权限提升功能的漏洞
* 拿到web权限后上传信息收集脚本到tmp目录下执行，例如linux-exploit-suggester，列出可能存在的内核漏洞

#### Suid提权

利用suid提权的核心就是运行root用户所拥有的有suid权限的文件，运行该文件的时候以root身份运行，获得了root权限，再借此执行/bin/bash得到root级别的shell

#### 自动任务提权

遇到root用户的定时任务，在自动任务中写入命令，即可获得root权限shell
