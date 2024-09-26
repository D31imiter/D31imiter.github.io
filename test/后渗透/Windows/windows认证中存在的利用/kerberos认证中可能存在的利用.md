### windows kerberos认证中可能存在的利用

#### Client-AS认证

`Client` 携带一个被自身 `NTLM Hash` 加密的身份信息 `AS Request` 到 `KDC` ，`KDC` 通过 `AD` 中对应的用户名提取 `Client` 的 `NTLM Hash` ，验证用户是否存在

`AS` 验证 `AS Request` 中 `Client info` 信息是否为发送请求的主机，`AS` 从 `AD` 提取 `Client` 的对应的 `NTLM Hash` 对 `AS` 中的 `Pre-authentication data` 数据解密，进行验证

```
基于Pre-authentication data认证机制，用户名存在和用户名不存在返回不同的数据。当对用户名枚举，找存在的用户名，可以使用kerbrute，以及nmap的利用脚本：
kerbrute userenum -dc [domain name] [username_list]
nmap -p[port] --script=krb5-enum-users --script-args krb5-enum-users.realm='[domain name]',userdb=[username_list],[domain name]
```

验证成功后，`KDC` 生成 `AS-Session-Key` ,返回两部分内容给 `Client`

1. `AS-Session-Key`（被 `Client NTLM Hash` 加密）
2. `TGT`：`AS-Session-Key` , `Client info` , `End Time`（被 `krbtgt NTLM Hash`加密）

`Client` 通过自己的 `NTLM Hash` 对第一部分解密后得到 `AS-Session-Key`，携带 `TGT` 进入第二步认证

#### Client-TGS认证

`Client` 发送请求：

1. `Client info` 以及 `Timestamp` ，也称 `Authenticator`（被 `AS-Session-Key` 加密）
2. `TGT`（已被 `krbtgt NTLM Hash` 加密，详见上）
3. `Client info` , `Server info` , `TimeStamp`

`KDC` 中的 `TGS` 收到请求后利用 `krbtgt NTLM Hash` 解密 `TGT` 得到 `AS-Session-Key` ，用 `AS-Session-Key` 解密 `Authenticator` 并验证

```
KDC并没有保存AS-Session-Key，KDC通过解密TGT得到AS-Session-Key，AS-Session-Key可以伪造
如果攻击者获取到了krbtgt NTLM Hash，就可以通过更改Cient info，Server info等参数来伪造任意的合法TGT，TGS被欺骗，返回攻击者想得到的任何票据，我们称其为黄金票据

mimikatz获取票据：
kerberos::golden /user:[administrator name] /domain:[domain name] /sid:[domain sid] /krbtgt: [krbtgt NTLM Hash] /ticket:[ticket name]
将票据注入到内存：
kerberos::ptt [ticket name]
```

然后生成新的临时随机密钥 `TGS-Session-Key` （也有人称 `Server Session Key`），发送回应，内容如下：

* `TGS-Session-Key`（被 `AS-Session-Key`加密）
* `Ticket`（被 `Server NTLM Hash` 加密），被加密的内容有：`TGS-Session-Key` , `Client info` , `End Time`

#### Client-Server认证

`Client` 收到回应使用保存的 `AS-Session-Key` 解密得到 `TGS-Session-Key`，发送请求，内容如下：

* `Client info` , `TimeStamp` （被 `TGS-Session-Key` 加密）
* `Ticket` （被 `Server NTLM Hash` 加密，详见上）

```
Server并不知道TGS-Session-Key，Server 通过解密Ticket得到TGS-Session-Key，TGS-Session-Key可以被伪造
唯一不知道的内容为Server NTLM Hash，如果攻击者获取到了这个值，Client可以伪造该服务的Ticket，我们称其为白银票据
```

`Server` 收到请求后，用自己的 `NTLM Hash` 解密得到 `Ticket` 中的 `TGS-Session-Key` ，再利用 `TGS-Session-Key` 解密第一部分，并根据请求的第一部分内容验证 `Ticket` ，认证成功后该票据会一直存在客户端内存中

```
攻击者攻陷一台主机时，即可转储出内存中的票据，再利用此进行横向移动

转储票据：
kerberos::list
kerberos::tgt
```
