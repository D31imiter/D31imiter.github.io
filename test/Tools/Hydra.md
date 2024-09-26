### Hydar

主要用来暴力破解ssh

```
hydra -L user.txt -P passwd.txt -o ssh.txt -vV ssh://ip -s 22

-l 指定单个用户
-L 指定用户字典文件
-p 指定单个密码
-P 指定密码字典文件
-o 把成功的输出到ssh.txt文件
-vV 显示详细信息
-s 指定其他端口 如果要修改默认22端口，可以使用 -s 参数
```
