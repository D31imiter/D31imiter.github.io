### gdb-debug

#### 运行

* start：运行到main函数
* run：直接运行
* starti：开始执行程序，在第一条指令处会停下来

#### 查看内存

1. info register（，i r）：查看寄存器
2. disassemble [address]：将address处内容反汇编为汇编代码显示
3. set disassembly-flavor intel：将AT&T汇编转为intel汇编

x/[n] [adress]： 按十六进制格式显示内存数据

```
x 按十六进制格式显示变量。
d 按十进制格式显示变量。
u 按十进制格式显示无符号整型。
o 按八进制格式显示变量。
t 按二进制格式显示变量。
a 按十六进制数据显示变量。

c 按字符格式显示变量。
f 按浮点数格式显示变量。

i 以反编译成的汇编代码显示数据
b 以1字节来显示
w 以4字节来显示
g 以8个字节显示
```

#### 断点

1. break *[address]：设置断点
2. info breakpoint：查看断点
3. disable [断点编号]：断点失效
4. enable [断点编号]：断点生效
5. delete [断点编号]：删除断点

#### 执行

1. ni：逐条执行，遇到函数步过，不进入内部
2. si：逐条执行，遇到函数进入
3. finish：进入到函数内部时直接执行到返回

#### 设置值

`set {[data type]}[adress]=0x4012DA`：将某处内存值设为，例如 `set {unsigned long long}[0x4012DA]=0x4012DA`

`set $rax=0x10`：设置寄存器的值

#### 联合tmux调试

开启tmux，Ctrl+B+%，开启两个终端

`set context-output /dev/pts/x` 将栈和寄存器等信息输出到伪终端x，方便查看

#### 其他debug

Radare2，Frida，DynamoRIO，PIN
