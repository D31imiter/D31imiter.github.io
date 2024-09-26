#### 在工作组环境中的认证

1. 客户端访问受保护的服务端需要输入服务器的用户名和密码进行验证，客户端在本地缓存服务器密码的NTLM Hash，向服务器发送TYPE 1 Negotiate消息，消息中包含了：明文用户名，需认证的主体，需要使用的服务，其他协商信息
2. 服务端收到消息后先判断本地账户中是否有TYPE1 中的用户名，如果有，服务端会返回自己能够支持和提供的服务内容，以及一个16位的随机challenge（服务器也会在本地缓存这个值），TYPE 2 challenge
3. 客户端收到回应后，使用步骤1缓存的服务器的NTLM Hash对challenge进行加密生成Response，然后将Response，用户名，challenge等组合得到Net-NTLM Hash，再将此Hash封装到TYPE 3 Authenticate 中发送到服务器
4. 服务端收到后用自己的NTLM Hash加密challenge与用户发来的数据比对是否一致，如果一致，认证成功

#### 在域中的认证

域中用户的NTLM Hash都储存在域控的NTDS.dit文件中，服务器自己无法完成认证，因此要与域控建立一个安全管道，通过域控完成最终的认证

1. 域用户登录客户端主机时，客户端会将用户输入的密码转化为NTLM Hash缓存，当用户访问受保护的服务端需要输入服务器的用户名和密码进行验证，客户端在本地缓存服务器密码的NTLM Hash，向服务器发送TYPE 1 Negotiate消息，消息中包含了：明文用户名，需认证的主体，需要使用的服务，其他协商信息
2. 服务端收到消息后先判断本地账户中是否有TYPE1 中的用户名，如果有，服务端会返回自己能够支持和提供的服务内容，以及一个16位的随机challenge（服务器也会在本地缓存这个值），TYPE 2 challenge
3. 客户端收到回应后，使用步骤1缓存的服务器的NTLM Hash对challenge进行加密生成Response，然后将Response，用户名，challenge等组合得到Net-NTLM Hash，再将此Hash封装到TYPE 3 Authenticate 中发送到服务器
4. 服务器收到TYPE 3消息后将此转发给域控
5. 域控收到TYPE 3 转发后，用TYPE 3中的用户名在NTDS.dit文件中找到对应的NTLM Hash，用此加密TYPE 3中的challenge，再与TYPE 3中的Response对比，一致则认证成功
6. 服务器根据域控返回的结果响应给客户端

#### 对NTLM认证的利用

1. 通过Responder截获用户认证请求，爆破Net-NTLM Hash得到用户NTLM Hash，对于NTLM v1级别的认证可以很容易的破解，NTLM v2级别的算法需要的算力很大
2. NTLM Relay，中继攻击，相当于伪造了服务端，诱使用户向恶意服务器发起认证，恶意服务器利用客户端发起的请求通过了服务端的认证

##### 获取
