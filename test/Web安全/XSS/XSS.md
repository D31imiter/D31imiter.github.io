### XSS

1. 类型：反射型，储存型，DOM型
   1. 反射型：用户在已登录账户时，点击了攻击链接，cookie被盗取
   2. 储存型：一般为论坛网站，输入的恶意html元素被储存到数据库中，正常用户访问网站时收到恶意脚本攻击
   3. DOM型：通过改写html元素写成的js脚本
   
   pikachu靶场：
   
   1. 反射型（get）：
   
      1. 输入aaa提交，F12查看网页源代码：
   
         (<p class="notice">who is aaa,i don't care!</p>)
   
         <p class="notice">who is aaa,i don't care!</p>
   
      2. 直接写入<script>alert(1)</script>，发现长度不够，观察刚刚提交的参数在url中显示，是get方式传递的参数，直接在url中提交，弹窗，有xss漏洞
   
   2. 反射型（post）：
   
      根据提示登录后输入<script>alert(document.cookie)</script>提交，获取cookie
   
   3. 储存型XSS：
   
      输入<script>alert(1)</script>发表评论，该元素直接被写入页面，每次打开这关都会弹窗
   
   4. DOM型XSS：
   
      1. 输入aaa提交，查看网页源代码发现：
   
         （<a href="aaa">what do you see?</a>）
   
         通过闭合引号和标签构造：输入   aaa"</a><a href="www.baidu.com" onclick="alert(1)
   
         <a href="aaa"</a><a href=&quot;www.baidu.com" onclick=&quot;alert(1)">what do you see?</a>
   
         无效
   
      2. 搜索what do you see，发现另一处代码：
   
         ~~~ javascript
         function domxss(){
             var str = document.getElementById("text").value;
             document.getElementById("dom").innerHTML = "<a href='"+str+"'>what do you see?</a>";
         }          
         ~~~
   
         输入的引号不转义输入的内容外有引号
   
      3. 在输入内容前加引号闭合掉即可：' onclick="alert('xss')">
   
   5. xss盲打：
   
      1. 输入asdsa，dasdas提交没有反应
      2. 查看提示，登陆后台，发现刚刚提交的数据
      3. 输入<script>alert(document.cookie)</script>语句提交
      4. 返回后台刷新，弹出cookie
   
   6. xss过滤：
   
      1. 输入<script>alert(1)</script>，就显示了一个<
      2. script标签被过滤，试试事件触发
      3. 输入</p>闭合前方标签，输入<p onclick="alert(1)">闭合后方</p>
      4. 点击弹窗
   
   7. xss之htmlspecialchars：
   
      htmlspecialchar(string ,flags,character-set,double_encode)：第一个参数为必选参数，表示待处理的字符串，第二个参数为可选参数，专门针对字符串中的引号操作，默认值：ENT_COMPAT，只转换双引号。ENT_QUOTS,单引号和双引号同时转换。ENT_NOQUOTES，不对引号进行转换。第三个参数为处理字符串的指定字符集。函数把预定义的字符转换为了HTML实体从而使xss攻击失效，但这个函数不会过滤双引号和单引号，只有设置了quotestyle规定如何编码双引号和单引号，才能过滤掉双引号和单引号。
   
      试一下单引号：aaa' onclick='alert(1)，弹窗
   
   8. xss之href输出：
   
      href 属性的值可以是任何有效文档的相对或绝对 URL，包括片段标识符和 JavaScript 代码段。如果用户选择了`<a>`标签中的内容，那么浏览器会尝试检索并显示 href 属性指定的 URL 所表示的文档，或者执行 JavaScript 表达式、方法和函数的列表。
   
      1. 试下<script>alert(1)</script>不行，然后不行查看源代码，符号都被编码了
      2. 在href属性中尝试输出，javascript:alert(1),弹窗
   
   9. xss之js输出：
   
      输入aaa提交，查看源代码发现：
   
      ~~~ javascript
      <script>
          $ms='aaa';
          if($ms.length != 0){
              if($ms == 'tmac'){
                  $('#fromjs').text('tmac确实厉害,看那小眼神..')
              }else {
                  alert($ms);
                  $('#fromjs').text('无论如何不要放弃心中所爱..')
              }
          }
      </script>
      ~~~
   
      输入的内容已经在script标签以内了，输入
   
      ~~~ 
      ';alert(1);//
      ';完成$ms的赋值，再输入alert（1），//用来注释后边的内容
      ~~~
   
      
   
      
   
      [XSS平台-XSS安全测试平台](https://xss.yt/index/user)
   
      在xss漏洞处写入该平台生成的项目代码即可得到部分权限
   

