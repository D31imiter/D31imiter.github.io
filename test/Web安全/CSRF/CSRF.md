#### CSRF

Cross-site request,跨站请求伪造,用户在授权登录了一个网站的情况下，点击了攻击链接，发送了一个针对已自己的授权网站的用户的攻击数据包，相当于攻击者借用了被攻击者的cookie完成的攻击，与xss漏洞不同的时是，xss通过处插入javascript代码document.cookie来读取先拿到cookie再实施的攻击。

进行CSRF攻击被攻击者需要点击攻击者制造的链接，有可能是攻击者向被攻击者发送的不明链接，也有可能是被攻击者已授权登录网站xss漏洞处的一个跳转或数据包的请求。

针对CSRF的防御有Token和Referer：

一般真实情况下用户不会自己修改数据包，所以

Referer：如果请求本身就是站内的，例如通过XSS攻击，可以实现绕过

Token：如果xss payload活得了Token，可以实现绕过





