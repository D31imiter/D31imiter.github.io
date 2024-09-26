### 关键名词

* Kerberos认证：每个域控制器都有一个 `krbtgt`的用户账户，是KDC的服务账户，用来创建票据授予服务（TGS）加密的密钥
* 域控制器（Domain Controller，DC）
* 密钥分发中心（Key Distribution Center，KDC）
* 账户数据库（Account Center，AD）；类似于SAM的数据库，储存所有Client用户的白名单，只有白名单中的Client才能成功申请TGT
* 身份验证服务（Authentication Service，AS）：为Client生成TGT的服务
* 入场券，认证票据（Ticket Granting Ticket，TGT）：入场券，通过入场券可以获得票据，是一种临时凭据的存在
* 票据发放服务（Ticket Granting Service，TGS）：为Client生成某个服务的票据
* 票据（Ticket）：Service Ticket，简称ST，网络对象互相访问的凭证
* 长期密钥，被hash加密的用户密钥（Master key | Long-term Key）：将本机密码做hash运算（NTLM）得到一个hash值，我们一般管这样的hash值为Master key
* 短期会话密钥，（Session Key | Short-term Key）：一种只在一段时间内有效的密钥

### 基本流程

1. Client携带账户信息向密钥分发中心（KDC）的身份验证服务（AS）发送想要访问Server A的请求，索要入场券（TGT），身份验证服务通过账户数据库验证Client用户的访问权，验证成功后返回TGT
2. Client携带入场券（TGT）请求密钥分发中心（KDC）的票据发放服务（TGS）发送想要访问Server A的请求，索要票据（Ticket），票据发放服务验证Client的入场券（TGT），具有Server A的访问权，返回票据（Ticket）
3. Client携带票据（Ticket）与服务器进行相互验证，成功后可以访问Server A，但无权访问其他服务器

### 具体流程

1. `Client` 携带一个被自身 `NTLM Hash` 加密的身份信息 `AS Request` 到 `KDC`，`KDC` 可以通过 `AD` 中对应的用户名提取 `Client` 的 `NTLM Hash`， `KDC` 验证用户是否存在于AD中

   * `AS Requset`大致内容：
     * `Pre-authentication data`，被 `Client`加密过的 `Timestamp`（防爆破），时间同步尤为重要
     * `Domain name\Client name`（`Client info`）
     * `Server Name`：KDC中TGS的 `Server Name`

   `AS`需要验证 `Client info `是否为本人，AS 需从 AD中提取 `Client `对应的 `NTLM Hash` 对 `Pre-authentication data` 进行解密，如果是一个合法的  Timestamp（时间差距合理），则可以证明发送方提供的用户名是存在于白名单中且密码对应正确的

   验证成功后会返回给Client一个 `AS Response`，主要包含两个部分：`Client`的 `NTLM Hash` 加密后的 `TGS Session Key`，和TGT，Client无法解密TGT，Client没有KDC 的hash

   * `TGT`大致内容（krbtgt的 `NTLM Hash`加密）
     * `Session Key`：第一部分的 TGS Session Key
     * `Domain name\Client`：（Client info）
     * `End time`：TGT到期的时间

   KDC此时生成一个 `Session Key`，使用 `Client`对应的 `NTLM Hash` 加密  `Session Key`，作为 AS数据，用于后续与TGS通讯，使用krbtgt的 `NTLM Hash`加密 `Session Key`和 `Client info`，生成TGT。Client通过自己的 `NTLM Hash` 对第一部分解密后得到 `TGS Session Key`，携带 `TGT`进入第二步
2. Client使用AS返回的TGS Session Key建立访问KDC中TGS服务的请求 `TGS Requset`，再将TGT连同请求一起发送的TGS服务

   * `TGS Request`大致内容：
     * Authenticator：（Client info + Timestamp）：通过TGS Session Key加密
     * TGT：TGS Session Key，Clinet info，End Time
     * Client info：Domain name/Client
     * Server info：Client试图访问的Server
     * Timestamp

   TGS收到TGS Request需要验证TGT与Authenticator被TGS Session Key加密，TGS服务并没有保存TGS Session Key，TGS使用自己的Master Key解密获取TGT中的TGS Session Key，进而解密Authenticator，验证客户端是否受信

   验证成功后，TGS返回一个TGS Response，包含两个消息：加密的Ticket，加密的Session Key

   * TGS Response大致内容：
     * Ticket：被Server 的Master Key加密
     * 通过TGS Session Key加密的 Server Session Key，用于Server Service 与 Client的通信使用
   * Ticket大致内容：
     * Server Session Key
     * Client info
     * End time ：Ticket到期时间

   Client收到TGS Response使用TGS Session Key解密得到Server Session Key 后进入第三步
3. Client通过第二步获得的Server Session Key创建用于证明自己自己就是Ticket所有者的Authenticator和时间戳，并用Server Session Key加密

   向服务器请求后，服务器用自己的Master Key解密Ticket，得到Server Session Key，使用Server Session Key解密Authenticator进行验证

   验证成功后返回给Client新的时间戳，使用Server Session Key加密

   Client通过缓存中的Server Session key解密服务器的返回信息，得到新时间戳并验证其是否正确，验证通过说明客户端可以信赖服务器，服务器向客户端提供相应的服务，并且该Ticket会一直存在客户端内存中
