# c++_IO

 ## cout类

1. ~~~ c++
   cout << argument; //重载运算符<<，打印参数
   ~~~

2. ~~~ c++
   cout << argument << argument;//运算符<<返回调用该运算符的引用，可以后接<<继续输出
   ~~~

3. cout的其他方法

   1. ~~~ c++
      cout.put('char');//打印该字符，返回该对象的引用
      ~~~

   2. ~~~ c++
      cout.write("char*",num);//打印从char*开始的num个字符，返回该对象的引用
      ~~~

4. 刷新输出缓冲区

   ~~~ c++
   cout<<flush<<endl;//flush刷新缓冲区，endl输出换行符到缓冲区刷新缓冲区
   ~~~

5. cout输出格式化

   1. 计数系统：被设置后一直生效

      ~~~ c++
      cout<<hex;//转换为十六进制
      cout<<dec;//转换为十进制
      cout<<oct;//转换为八进制
      ~~~

   2. 调整字段宽度：被设置后只影响将显示的下一个项目，然后将字段恢复为默认值

      ~~~ c++
      cout<<width(num);//将字段的宽度设置为num，默认向右对其
      ~~~

   3. 填充字符：设置后一直有效

      ~~~ c++
      cout.fill('char');//用该字符填充字段中未被使用的部分
      ~~~

   4. 设置浮点数的显示精度：设置后一直有效

      ~~~ c++
      cout.precision(num);//将精度设置为num，该处精度为总位数
      ~~~

   5. setf() 方法：设置该类的属性，设置后一直有效

      1. ~~~ c++
         fmtflags set(fmtflags);
         cout.setf(ios_base::boolalpha);//输入和输出bool值为”true“和”false“，不为0和1
         cout.setf(ios_base::showbase);//对于输出使用c++基数前缀（八进制（0），十六进制（0x））
         cout.setf(ios_base::showpoint);//打印末尾的0和小数点
         cout.setf(ios_base::uppercase);//对于十六进制输出，使用大写字母
         cout.setf(ios_base::showpos);//在正数前面加上+，基数为十才使用加号，c++将十六进制和八进制视为无符号的，因此对他们无需使用符号
         ~~~

      2.  ~~~ c++
          fmtflags set(fmtflags,fmtflags);
          ~~~

         第一参数为所需设置的fmtflags值，第二参数指出要清除第一个参数中的哪些位

         |                       | set(fmtflags,fmtflags)的参数 |                                |
         | --------------------- | ---------------------------- | ------------------------------ |
         | 第一个参数            | 第二个参数                   | 含义                           |
         |                       | ios_base::dec                | 使用基数10                     |
         | ios_base::basefield   | ios_base::oct                | 使用基数8                      |
         |                       | ios_base::hex                | 使用基数16                     |
         |                       | ios_base::fixed              | 使用定点计数法                 |
         | ios_base::floatfield  | ios_base::scientific         | 使用科学计数法                 |
         |                       |                              |                                |
         |                       | ios_base::left               | 使用左对齐                     |
         | ios_base::adjustfield | ios_base::right              | 使用右对齐                     |
         |                       | ios_base::internal           | 符号或基数前缀左对齐，值右对齐 |

         定点计数法及科学计数法：

         ~~~ c++
         int main()
         {
         	cout.setf(ios_base::fixed, ios_base::floatfield);
         	cout << "定点计数法：" << 12.345 << endl;
         	cout.setf(ios_base::scientific, ios_base::floatfield);
         	cout <<"科学计数法："<< 12.345 << endl;
         	return 0;
         }
         /*
         output：
         定点计数法：12.345000
         科学计数法：1.234500e+01
         */
         int main()
         {
         	cout.precision(3);
         	cout.setf(ios_base::fixed, ios_base::floatfield);
         	cout << "定点计数法：" << 12.345 << endl;
         	cout.setf(ios_base::scientific, ios_base::floatfield);
         	cout << "科学计数法：" << 12.345 << endl;
         	return 0;
         }
         /*
         output：
         定点计数法：12.345
         科学计数法：1.235e+01
         */
         ~~~

         注意：精度让默认的浮点显示总共显示几位，而定点模式和科学计数法只显示几位小数

         ~~~ c++
         cout.setf(0,ios_base::floatfield);
         cout.unsetf(ios_base::floatfield);
         //设置为默认模式
         ~~~

   6. 标准控制符：

      ~~~ c++
      cout<<Standard control character;//通过<<打开该选项，上述模式都可
      ~~~

   7. 头文件<iomanip>

      ~~~ c++
      setprecesion(arg);//指定精度的整数参数
      setfill(arg);//指定填充字符的char参数
      setw(arg);//指定字段宽度的整数参数
      ~~~


## cin类

1. ~~~ c++
   cin>>argument;//输入一串字节流，>>方法从字节流读取对应的数据给参数
   ~~~

2. cin>>检查输入：遇到符合类型的字节流时，读取，当遇到不符合类型的字符或空格、水平制表符、换行符时停止读取，留下字节流在缓冲区中等待处理

3. cin流状态：

   isostate :真实类型为int，流状态，有goodbit（0），eofbit（1），failbit（2），badbit（4）四种
   
   | 成员                     | 描述                                                         |
   | ------------------------ | ------------------------------------------------------------ |
   | eofbit                   | 如果到达文件尾，则被设置为1                                  |
   | badbit                   | 如果流被破坏则设置为1，例如，文件读取错误                    |
   | failbit                  | 如果输入操作未能读取预期的字符或输出操作没有写入预期的字符，则设置为1 |
   | goodbit                  | 另一种设置流状态为0的方法                                    |
   | good（）                 | 如果流可以使用（所有的位）都被清除，则返回true               |
   | eof（）                  | 如果eofbit被设置，则返回true                                 |
   | bad（）                  | 如果badbit被设置，则返回true                                 |
   | fail（）                 | 如果failbit或badbit被设置，则返回true                        |
   | rdstate（）              | 返回流状态                                                   |
   | exceptions（）           | 返回一个位掩码 ，指出哪些标记导致异常被引发                  |
   | exceptions（iostate ex） | 设置哪些状态将导致clear（）引发异常；例如，ex是eofbit，如果eofbit被设置，clear（）将引发异常 |
   | clear（iostate s）       | 将流状态设置为s；s的默认值为0（goodbit）；如果（restate（）& exceptions（））！= 0，则引发异常basic_ios::failure |
   | setstate（iostate s）    | 调用clear（rdstate（）\| s）。这将设置与s中设置的位对应的流状态位，其他流状态位保持不变 |
   
   1. 如果输入操作未能读取预期的字符或输出操作没有写入预期的字符，则设置为1，未能读取的字符被留在缓冲区里，由于流的状态依旧为1（failbit），针对于流操做不能进行，需要cin.clear(ios_base)。
   
      由于还有不匹配输入在缓冲区里，该流依旧无法使用，可以使用isspace（）函数
   
   2. 调用判断该流类型的函数返回的是bool值，1代表开启，0代表关闭。而流状态goodbit（0），eofbit（1），failbit（2），badbit（4)有对应的值，流状态所代表值之和就是rdstate（）的返回值
   
   3. 一旦有错误流goodbit就被关闭（cin.good()==0)，该流不能被使用
   
   4. clear（iostate）设置该流为1，设置其他流为默认值0；setstate（iostate）仅仅设置该流为1，不影响其他iostate
   
      特例：一旦badbit被设置为1，failbit一定也为1，其他流状态不确定
      
   5. exceptions（）：
   
      1. 有参的函数用来设置该异常将被触发，仅能设置一种，eofbit或failbit或badbit
      2. 无参的函数用来返回被设置的异常
   
   6. （restate（）& exceptions（））！= 0，三种异常流状态，用二进制表示为001，010，100，rdstate（）返回的是流状态之和，如果一旦异常被设置，与该流rdstate（）返回值按位与，rdstate（）和  exceptions（）有一位都为1则说明异常和流状态相符，引发异常
   
4. 其他istream方法：

   1. ~~~ c++
      cin.get()//无参的get，返回读取的字符的ascall码值，类型为int
      cin.get(char);//从缓冲区读取一个字符，空格，换行符，制表符都可以读取
      cin.getline(char*，n);//同getline
      cin.getline(char*,n,char);//同getline
      ~~~

   2. ~~~ c++
      cin.geline(char*，n);//从缓冲区读取字符n个字符到字符串包括末尾的换行符，getline可以读取水平制表符，空格
      cin.getline(char*,n,char);//char指定用作分界符的字符，只有两个参数的版本将用换行符作为分界符
      注意：读取到分界符字符时，都将该字符视为\0储存到数组中。get和getline的主要区别在于getline是读取一行，包括结尾的换行符；get不读取结束符，把结束符留在输入流中，下一次读取时，首先读取的是上一次留下的分界符字符 
      ~~~

   3. ~~~ c++
      cin.ignore(num,char);//读取丢弃接下来的num个字符，如果遇到char或者丢弃完num个字符就会停止读取
      ~~~

   4. ~~~ c++
      cin.read(字符数组,num);//读取num个字符到字符数组中，单纯读取，不会在末尾加上\0,是为文件读取设计的函数
      ~~~

   5. ~~~ c++
      cin.peak();//返回输入中的下一个字符，但不抽取输入流中的字符，如果输入流中没有任何字符，就等待输入
      ~~~

   6. ~~~ c++
      cin.gcount();//返回最后一次cin类方法的抽取的字符数，非格式化，意味该方法不是>>
      ~~~

   7. ~~~ c++
      cin.putback(char);//将该字符放入缓冲区的第一个字符，也就是下一条读取语句的第一个字符
      ~~~



