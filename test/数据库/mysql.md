
### 登录方式

1. 进入到bin目录下
2. mysql -uroot -p（password：root）
3. -- comment(mysql注释)

### 数据库操作

1. show databases;(显示所有数据库)

   show databases like '匹配模式';

   ex:

   1. 获取以my开头的所有数据库，'my%';
   2. 获取以my开头后面第一个字母不确定，最后为database的数据库，'m_database';
   3. 获取以database结尾的数据库，'%dadtabase';
2. create database database_name [库选项];(创建数据库）

   create database database_name charset gbk/utf8 collate gbk/utf8;

   1. 库选项：数据库的相关属性
   2. 字符集：charset 字符集（gbk，utf8）
   3. 校对集：collate 校对集
3. show create database database_name;(显示创建数据库语句)
4. use database_name;(进入到对应的数据库下)
5. alter database database_name charset (=) 字符集(gbk,utf8);(修改库选项)
6. drop database database_name;(删除数据库)

### 表操作

1. create table table_name(字段名 字段类型[字段属性]，字段名 字段类型[字段属性]......) [表选项];

   1. create table database_name.table_name(......);
   2. 进入到对应数据库，create table table_name(......);

   表选项：

   1. engine：储存引擎 innodb/myisam
   2. 字符集：charset 字符集（gbk，utf8）只对自己当前表有效，权限比数据库高
   3. 校对集：collate 校对集
2. create table new_table_name like table_name;(复制已有表结构,仅复制表结构,如果表中有数据则不复制)
3. show tables;(查看所有表，同样有匹配模式)
4. desc table_name/describe table_name/show columns from table_name;(显示数据表结构)
5. show create table table_name;(查看创建数据表时的语句)
6. drop table table_name1 table_name2.....;(删除数据表结构)

### 设置表属性

1. alter table table_name 表选项 值;

   alter table student charset utf8;
2. rename table old_table_name to new_table_name;


### 操作字段

1. alter table table_name add 新字段名 列类型 列属性;(新增字段名，默认放在最后)

   first：第一个字段

   after：after 字段名，放在某个具体的字段后
2. alter table 表名 change 旧字段名 新字段名 字段类型;(修改字段名)
3. alter table 表名 modify 字段名 字段类型;(修改字段 类型)
4. alter table table_name drop 字段名;(删除字段)

### 插入操作

1. insert into table_name (字段名,...) values(内容,...);(内容与字段名一一对应)
2. insert into table_name values(内容,...)，(内容,...)，(内容,...);(值列表必须对应表结构，插入多个内容)


### 查询字段

1. select distinct/all 列名 from 表名 as 别名 distinct/all 列名 from 表名 as 别名 where 条件判断 group by having

   1. 子查询：从另一张表的结果再次查询，select * from (select 列名 from 表名) as 新表名
   2. group by 只会保留一条记录，通过使用聚合函数来实现统计，多次使用group by实现多分组
   3. having使用在group by子句之后，可以对

   联合查询：（select order by语句）union（select order by语句）使用order by必须加括号
2. delete from table_name (where条件);(如果没有where条件则自动删除给表所有内容)

   注意：自增长偏移量不会充值，需要使用truncate table_name 清空表中所有数据以及重置自增长偏移量
3. update table_name set 字段名 = 新值(where 条件) limit;(更新数据，如果没有where条件，那么所有的表中对应的那个字段都会被修改成统一值，limit用来限制修改的行数)

### 字符集

1. cmd使用gbk字符集，mysql服务段默认utf-8字符集，需要将全局变量character_set_client，character_set_results，设置成相同字符集才能显示中文
2. 由于cmd默认使用gbk，最好改服务端，也就是变量character_set_results

### 数据类型

1. 整形：

   1. tinyint：一个字节大小
   2. mediumint：三个字节
   3. int：四字节
   4. bigint：八字节

   设置无符号属性在类型后加unsinged，设置显示长度在类型后加（length）

   注意：显示长度指的是可以到达指定的长度，但是不会自动满足到指定长度，如果想要固定该显示长度，需要加一个zerofill属性

   zerofill会使该数字左侧填充0，填充后长度为该类型的所能表示的最大值位数
2. 浮点型：

   1. 单精度浮点型：float，4字节
   2. 双精度浮点型：double，8字节
   3. 定点型：每9个数会多个分配四个字节，小数和整数是分开储存的

      decimal（M，D）：M表示总长度最大不能超65；D代表小数部分长度，最大不能超30

   对于浮点数可以四舍五入，定点数会报错
3. 时间日期类型：

   1. Date：日期类型，3字节，Y-M-D，表示的范围：1000-01-01到9999-12-12，初始值为0000-00-00
   2. Time：时间类型，3字节，H-M-S，表示的范围：-838：59：59到838：59：59，描述时间段

      可以使用 day H-M-S，day*24和并到小时数中
   3. Datetime：将Date和Tim合并，八字节
   4. Timestamp：时间戳不能为空，未指定插入数据时为当前时间，更新该记录时未指定插入数据时为当前时间
   5. Year：可以输入四位数插入，也可以输入两位数插入，输入两位数时：0-69前加20，70-99时前加19
4. 字符类型：

   1. char：char(length)，Length代表字符数，系统会分配固定空间，Length为0-255
   2. varchar：varchar(Length)，**变长字符**，Length代表字符数，Length：0-65535

      varchar要记录长度，自动分配空间

   注：char查询数据的效率要高于varchar

   一般长度超过255字符，使用text数据类型
5. Text数据类型：

   有四种text类型代表不同的长度，无需可以选择，系统会自动分配
6. Blog：二进制数据类型，图片或音乐的存储方式，一般不像这样储存，一般会使用链接指向资源
7. Enum：枚举类型，插入该列的数据必须是枚举中的一个；枚举存的下标从1开始，枚举用来规范数据
8. Set：集合数据类型，将多个数据选项同时保存的数据类型，本质是将指定的项按照对应的而二进制位来进行控制，1表示被选中，0表示不被选中，语法：set（'1','2','3'）

   1字节8选项，2字节16选项，最多有64个选项

   1. 插入的数据只与set类型有关
   2. 储存时是按颠倒的二进制转化为十进制储存的
   3. 插入时可以使用数字插入

### 列属性

1. null属性：该字段是否可为空
2. default：默认值，如果不进行该字段的插入则用默认值填充，也可使用default关键字触发
3. comment：提供列的描述
4. primary key：一张表中有且只有一个字段，值具有唯一性

   复合主键：有多个列具有主键属性，只要一个主键列中的字段不同时，都可以插入数据

   增加方式：创建表时在列名后加primary key，或primary key（列名）

   删除方式：alter table table_name drop primary key

   主键冲突处理方式：

   1. insert into table_name (字段列表) values(值列表) on duplicate update 字段 = 更改的主键新值
   2. replace into (字段列表) values(值列表)
5. auto_increment：自增长，不指定数据时按照上一个数据+步长插入，自动触发，系统中有初始值和步长变量(auto_increment_increment[步长],auto_increment_offset[偏移量])

   修改变量：alter table table_name auto_increment=10，表中的auto_increment是下一个不指定数据给出的自增长值

   删除自增长：alter table table_name modify column type;

   1. 一个表有一个自增长
   2. 给定数据不会触发自增长
6. unique key：保证字段唯一性，唯一键允许为null

   删除唯一键：alter table table_name drop index 唯一键名字（默认为列名）

   复合唯一键：同主键，但可以为null
