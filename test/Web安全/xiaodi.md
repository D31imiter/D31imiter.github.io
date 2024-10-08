# Total

## Day1

1. 例如www.baidu.com域名，com，org，cn为顶级域名，baidu等为一级域名，一级域名可以购买来使用，使用者自由划分其下的二级域名，如www，news等
2. DNS域名解析，将域名解析为IPV4地址，本地DNS服务器->根DNS服务器->顶级域（TLD）DNS服务器->权威DNS服务器，通过UDP协议实现

## Day2

Request请求数据包

1. 请求行：请求类型，请求资源类型，协议版本和类型

   请求方法，URL，HTTP版本
2. 请求头：一些键对值

   Host：主机名，域名地址，端口号

   Accept：用户可接受的文件格式，服务器可以根据此决定发送的文件格式

   User-Agent：客户浏览器名称

   Accept-language：浏览器可以接受的语言种类

   connection： tcp链接是否可持续

   Referer：表明产生请求的网页URl，跟踪Web请求是从什么网站来的

   Accept-Charset：字符编码类型

   X-Forwarded-For: 形式为client, proxy1, proxy2，用于获取真实IP
3. 空行：请求头与请求体用一个空行隔开
4. 请求体：要发送的数据（一般post提交会使用）id=1&1=1

Response返回数据包数据格式

1. 状态行：协议版本，数字形式的状态代码
2. 响应头标服务器类型，日期，长度，内容类型等
3. 空行：响应头和响应体之间通用空行隔开
4. 响应数据浏览器会将实体内容中的数据提取出来，生成相应的页面

HTTP响应码：

1xx：信息，请求收到，继续处理

- `100(继续)`请求者应当继续提出请求。服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。
- `101(切换协议)`请求者已要求服务器切换协议，服务器已确认并准备切换。

2xx：成功，行为被成功的接受

- 200(成功)服务器已成功处理了请求。通常，这表示服务器提供了请求的网页。
- 201(已创建)请求成功并且服务器创建了新的资源。
- 202(已接受)服务器已接受请求，但尚未处理。
- 203(非授权信息)服务器已成功处理了请求，但返回的信息可能来自另一来源。
- 204(无内容)服务器成功处理了请求，但没有返回任何内容。
- 205(重置内容)服务器成功处理了请求，但没有返回任何内容。
- 206(部分内容)服务器成功处理了部分 GET 请求。

3xx：重定向，为了完成请求，必须进一步执行的动作

- 300(多种选择)针对请求，服务器可执行多种操作。服务器可根据请求者(user agent)选择一项操作，或提供操作列表供请求者选择。
- 301(永久移动)请求的网页已永久移动到新位置。服务器返回此响应(对 GET 或 HEAD 请求的响应)时，会自动将请求者转到新位置。
- 302(临时移动)服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
- 303(查看其他位置)请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。
- 304(未修改)自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。
- 305(使用代理)请求者只能使用代理访问请求的网页。如果服务器返回此响应，还表示请求者应使用代理。
- 307(临时重定向)服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。

4xx：客户端错误

- 400(错误请求)表示客户端请求的语法错误，服务器无法理解，例如 url 含有非法字符、json 格式有问题。
- 401(未授权)请求要求身份验证。对于需要登录的网页，服务器可能返回此响应。
- 402表示保留，将来使用。
- 403(禁止)表示服务器理解请求客户端的请求，但是拒绝请求。
- 404(未找到)服务器无法根据客户端的请求找到资源(网页)。
- 405(方法禁用)禁用请求中指定的方法。
- 406(不接受)无法使用请求的内容特性响应请求的网页。
- 407(需要代理授权)此状态代码与 401(未授权)类似，但指定请求者应当授权使用代理。
- 408(请求超时)服务器等候请求时发生超时。
- 409(冲突)服务器在完成请求时发生冲突。服务器必须在响应中包含有关冲突的信息。
- 410(已删除)如果请求的资源已永久删除，服务器就会返回此响应。
- 411(需要有效长度)服务器不接受不含有效内容长度标头字段的请求。
- 412(未满足前提条件)服务器未满足请求者在请求中设置的其中一个前提条件。
- 413(请求实体过大)表示响应实在太大。服务器拒绝处理当前请求，请求超过服务器所能处理和允许的最大值。
- 414(请求的 URI 过长)请求的 URI(通常为网址)过长，服务器无法处理。
- 415(不支持的媒体类型)请求的格式不受请求页面的支持。
- 416(请求范围不符合要求)如果页面无法提供请求的范围，则服务器会返回此状态代码。
- 417(未满足期望值)在请求头 Expect 指定的预期内容无法被服务器满足(力不从心)。
- 418表示我是一个茶壶。超文本咖啡馆控制协议，但是并没有被实际的 HTTP 服务器实现。
- 420表示方法失效
- 422表示不可处理的实体。请求格式正确，但是由于含有语义错误，无法响应。

5xx：服务器错误

500(服务器内部错误)服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。

- 501(尚未实施)服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。
- 502(错误网关)服务器作为网关或代理，从上游服务器收到无效响应。
- 503(服务不可用)服务器目前无法使用(由于超载或停机维护)。通常，这只是暂时状态。
- 504(网关超时)服务器作为网关或代理，但是没有及时从上游服务器收到请求。
- 505(HTTP 版本不受支持)服务器不支持请求中所用的 HTTP 版本。

原文链接：https://blog.csdn.net/ChineseSoftware/article/details/123177081

## Day3

搭建平台脚本：ASP，PHP，ASPX，PY，JS，JAVAWEB

域名IP目录解析：IP网站根目录，可以扫到更多文件

文件后缀解析：通过设置解析可以使一个文件以不同的形式解析

常见测试中的安全防护：限制IP访问，黑白名单

Web后门与文件权限：设置IS来宾用户的权限防止后门植入，但一般根目录有执行权限，可以将后门植入到根目录

## Day5

1. 判断操作系统，大小写不敏感为Windows，敏感为linux或其他。nmap扫描。

   操作系统有漏洞。
2. 数据库：Access，mysql，Oracle

   Asp+Access, Php +Mysql, aspx +mssql, jsp + oracle/mssql,python+ mongodb

   1、mysql 默认端口为：3306	 

   2、sqlserver 默认端口号为：1433

   3、oracle 默认端口号为：1521

   4、DB2 默认端口号为：50000

   5、PostgreSQL 默认端口号为：5432

## Day6

1. MD5：有16位与32位两种加密方式，加密后的密文由0-9和a-z组成，一般不可逆，[md5在线解密破解,md5解密加密 (cmd5.com)](https://www.cmd5.com/)可以破解一定密文
2. SHA：Secure Hash Algorithm，SHA-1可以生成一个被称为消息摘要的160位（20字节）散列值，散列值通常的呈现形式为40个十六进制数，SHA-256可以生成一个被称为消息摘要的256位（32字节）散列值，散列值通常的呈现形式为64个十六进制数，（SHA512）以此类推。英文字母可大写可小写
3. 时间戳：从1970年1月1日至今的总秒数，一串数字
4. URL：URL 字符转义的方法是，在这些字符的十六进制 ASCII 码前面加上百分号（`%`）

   EX（%20）：20->32,ascall码值为32，的是（‘空格’）

   ```html
   !：%21	#：%23	：%24	&：%26	'：%27	(：%28	)：%29	*：%2A	+：%2B	,：%2C	/：%2F	:：%3A
   ;：%3B	=：%3D	?：%3F	@：%40	[：%5B	]：%5D
   ```
5. BASE64（一种编码）：

   1. 随明文变长，密文也随之边长，长度不固定。
   2. 密文中大小写英文字母，0-9混合出现，区分大小写。
   3. 密文末尾一般带有（’=‘）；
6. Unescape：%u后加四位数字代表一个一个数字或字母
7. AES：加密前的编码：base，16进制，二次解码。模式，填充位数密匙，偏移量。解密时偏移量和密匙必须知道。
8. DES：同AES，但随明文增长而增长

## Day7

1. CDN：缓存服务器发送的内容，收到请求时如果有资源则CDN节点发送，否则请求服务器
2. CDN绕过：
   1. 超级Ping，Fofa，shodan搜索引擎
   2. 从子域名突破，主站的访问量高有CDN节点，子域名没有CDN，ping子域名（如果同个IP）
   3. 邮件查询
   4. 国外地址请求，无CDN节点
   5. DNS历史记录，位申请CDN之前的IP
   6. m.xxx.xxx,移动端网站
   7. 工具扫全网

## Day8

1. 有无cdn，先找到服务器IP
2. 寻找网站源码
3. 是否是app，pc端或者移动端
4. 第三方的信息收集，whois的备案，github上是否有开源
5. 通过的第三方的应用，数据库，管理平台等
6. 找各种服务接口，是支付接口还是专门用来储存图片的
7. 微信公众号，社工应用的信息收集

旁注：同服务器不同站点

C段：同网段不同服务器不同站点

WAF：wafw00f工具识别waf类别

## Day11

- SQL注入：页面对参数没有过滤，恶意的语句可以被执行，可以拿到数据库内容，如果可以拿到管理员用户的账号密码，可以拿到后台的权限
- 文件上传：文件上传漏洞是指用户上传了一个可执行的脚本文件，并通过此脚本文件获得了执行服务器端命令的能力。
- XSS跨站：一种代码注入攻击，攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行，分三种类型：反射型，储存型，DOM型
- 文件包含：随着网站业务的需求，程序开发人员一般希望代码更灵活，所以将被包
  含的文件设置为变量，用来进行动态调用，但是正是这种灵活性通过动态变
  量的方式引入需要包含的文件时，用户对这个变量可控而且服务端又没有做
  合理的校验或者校验被绕过就造成了文件包含漏洞。
- 反序列化：当程序在进行反序列化时，会自动调用一些函数，例如__wakeup(),__destruct()等函数，但是如果传入函数的参数可以被用户控制的话，用户可以输入一些恶意代码到函数中，从而导致反序列化漏洞。
- 代码执行：用户输入的数据，被当做后端代码进行执行
- 逻辑安全：修改数据包，比如修改物品的价格达到0元购的目的
- 未授权访问：未授权访问漏洞可以理解为需要安全配置或权限认证的地址，授权页面存在缺陷导致其他用户可以直接访问从而引发重要权限可被操作、数据库或网站目录等敏感信息泄露。
- CSRF：通过修改cookie，请求数据包的请求者达到一以其他用户的名义购买商品，转账
- SSRF：服务器端请求伪造，攻击的目标是从外网无法访问的内部系统。Web应用脚本提供了从其他服务器应用获得数据的功能，但没有对目标地址进行过滤和限制
- 目录遍历：程序在实现上没有充分过滤用户输入的…/之类的目录跳转符，导致恶意用户可以通过提交目录跳转来遍历服务器上的任意文件。
- 文件读取：攻击者通过一些手段可以读取服务器上开发者不允许读到的文件。主要读取的文件是服务器的各种配置文件、文件形式存储的密钥、服务器信息（包括正在执行的进程信息）、历史命令、网络信息、应用源码及二进制程序。
- 文件下载：任意漏洞下载是因为一般的网站提供了下载文件功能，但是在获得文件到下载文件的时候并没有进行一些过滤，恶意用户就可以利用这种方式下载服务器的敏感文件，对服务器进行进一步的威胁和攻击。
- 命令执行：服务器端没有对客户端用户输入的命令进行过滤，导致用户可以通过任意拼接系统命令，使服务器端成功执行任意系统命令。
- XXE安全：XXE漏洞发生在应用程序解析XML输入时，没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站、发起dos攻击等危害。XXE漏洞触发的点往往是可以上传XML文件的位置，没有对上传的XML文件进行过滤，导致可上传恶意XML文件。
