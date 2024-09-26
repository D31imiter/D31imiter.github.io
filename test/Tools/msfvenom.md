#### msfvenom

-l ：列出所有攻击模块

-p ： 指定使用的payload 

-f ：指定输出格式，也就是后缀名

-e：指定使用的编码器

-i ：指定payload的编码次数

-a ：指定目标机器的架构，例如x86 还是 x64 还是 x86_64

-o ： 输出生成文件的路径

-b ： 指定坏字符，规避坏字符

-s ： 指定payload最大长度

-x ：指定一个可执行文件作为模板，将payload嵌入其中，也就是捆绑木马

-k, --keep ：保护模板程序的动作，注入的payload作为一个新的进程运行

--platform < platform> 指定payload的目标平台
