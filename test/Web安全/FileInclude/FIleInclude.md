#### 文件包含

include,include_once,require,require_once,文件包含函数会将包含的文件中的内容以php代码执行，include类的函数会产生警告，脚本继续运行，而require类的函数会在出错时产生E_COMPILE_ERROR级别的错误，停止运行

当url处有类似文件名的参数时会产生文件包含漏洞，可以随意更改参数来访问服务器本地文件，allow_url_include开启的情况下可以访问远程文件，危害更大

攻击者也可利用php伪协议，进行读写php代码

1. file://     访问本地文件

2. http://    访问 HTTP网址

3. php://      输入输出流访问

   1. url?filename=php://filter/read=过滤器1|过滤器2/resource=要读取的文件的相对当前网页路径

      过滤器：

      1. 字符串过滤器

         1. string.rot13：字符右移13位
         2. string.toupper：将所有字符转换为大写
         3. string.tolower：将所有字符转换为小写
         4. string.strip_tags：去除内容中的标签

      2. 转换过滤器：convert.base64-encode和convert.base64-decode

         防止直接写出php代码直接执行导致编译后看不到源码

   2. url?filename=php://input    post参数      allow_url_include:on

      将post参数当作php代码执行，一般直接写入一句话木马不行，需要新建文件写入

       ex：

      ~~~php
      <?php fwrite(fopen("not_shell.php","w+") ,'<?php @eval($_POST["cmd"]); ?>');?>
      ~~~

4. data://text/plain,php代码      allow_url_fopen:on & allow_url_include :on   执行php代码