#### 利用原理

格式化输出函数的每一个指定的格式需要对应一个参数，而当不指定格式的参数时，格式化输出函数将默认传参，x86的默认传参为从右至左依次入栈，而x64默认为rdi，rsi，rdx，....，剩余入栈，这就导致程序没有指定值，而导致传入了未知的值

当出现 `printf(str)`，传入恶意的字符串，执行格式化时printf将%类的参数执行，形成漏洞

漏洞造成攻击者控制str的值，即控制栈上的内容，即格式化字符串函数的参数

##### 泄露内存

###### 泄露栈内存

当长度无限制时，通过填充%p来泄露栈上内容，当有长度限制时，通过参数 `%[var]$[fmt type]`来获取指定偏移的内容

`%[var]$[fmt type]`：var为格式化字符串的第(var+1)个参数，fmt type决定输出内容

```
格式化字符串传参时就会有一个参数，printf(str)，第一个参数为str，所以对应的操作数会加1
即%1$p时：x64打印rsi寄存器的值，x86打印（str+4）处的值，str为字符指针
注意没有%0$[fmt type]这种操作
同理，如果打印%p%p%p，x64打印先rsi寄存器的值,此后递增，x86同理
```

`%7$p`：将第七个参数位置处的值当作地址输出

`%7$d`：将第七个参数位置处的值当作整数输出

###### 泄露任意地址内存

试想 `%[var]$s`，将var偏移处的值当作字符串指针，输出这个字符串，如果控制var我们想要泄露内存的地址，就可以实现泄露任意地址内存

#### 任意地址写

%hhn(1byte)，%hn(2byte)，%n(4byte)，%lln(8byte)等参数

将之前已经打印的字符个数赋值给参数，参数是一个指针

当执行 `printf("123456%n",&count)`时，将值赋值给count变量

##### 栈覆写

同 `%[var]$[fmt type]`，构造 `%[var]$n`，执行%n的特性，将已经输出的字符数写入var偏移处的指针变量

#### fmtstr

pwntools工具库中集成了一个格式化字符串攻击载荷生成器

```
fmtstr_payload(offset, writes, numbwritten=0, write_size='byte', write_size_max='long', overflows=16, strategy="small", badbytes=frozenset(), offset_bytes=0)
```

* offset：第一个格式化程序的偏移量
  寻找方法：

  ```
  在x64架构，格式化的第一个参数是对应字符串，其余参数为rsi,rdx,rcx,r8,r9,第六个参数为栈顶的内容，
  偏移量：5+((str_addr-$rsp)/8)+1
  x86同理
  ```
* write：要写入的内容，字典类型，如 `{addr1:value1, addr2:value2}`
* numbwritten=0：已经由printf写入的字节数
* write_size='byte'：指定逐byte/short/int写
