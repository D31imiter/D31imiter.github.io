#### 安装

mimikatz源码：[GitHub - gentilkiwi/mimikatz: A little tool to play with Windows security](https://github.com/gentilkiwi/mimikatz)

mimikaze可执行文件：[GitHub - ParrotSec/mimikatz](https://github.com/ParrotSec/mimikatz)

#### 使用

提升权限：`privilege::debug`

#### 常用模块

##### sekurlsa

1. msv：抓取用户NTLM Hash
2. kerberos：列出kerberos认证中的相关凭据
3. tspkg：TSPKG代表"Terminal Services Security Package"，是Windows中用于远程桌面协议的安全包，该模块可列出远程桌面服务登录的相关凭据
4. logonpassword：列出所有提供者的凭据
5.
