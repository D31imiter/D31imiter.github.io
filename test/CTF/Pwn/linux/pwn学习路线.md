### pwn_usermode

出题者给出一个二进制文件，通过与文件交互，利用漏洞得到shell

#### 环境配置

kali系统和ubuntu系统都可以

常用工具为IDA，gdb配好pwndbg插件，一个好用的python文本编辑器，python和c的环境，python的pwntools包（自带checksec，用来查看elf保护机制的），tmux

1. IDA PRO网上找一个
2. 插件有很多，pwndbg好用一些，在家目录的.gdbinit中启动pwndbg插件就配置完了，网上有详细配置教程
3. 文本编辑器想用什么都可以，我自己用的pycharm，占用内存大，接下来的例子也讲的是pycharm下的调试

#### 大致流程

拿到一个elf文件，首先取去查看这个文件的版本64位还是32位，checksec看保护机制，题目中给的什么版本的libc（有时通过远程泄露一些函数地址得知libc版本）然后patchelf换对应版本，然后ida静态分析定位到漏洞的位置，然后动态调试写exp

##### 调试技巧

1. 静态分析，将文件用ida打开，常用的功能
   * `F5`反编译汇编代码为c伪代码
   * `Tab`在反编译的c代码和汇编代码之间切换
   * `G`跳转到某一地址
   * `shift+F12`字符串搜索
   * `Esc`跳转回上一个函数
2. 动态调试
   1. gdb：通过tmux开启两个终端（按了 `ctrl+b `后按 `%`，tmux其他使用具体去查），在其中一个终端内输入tty查看，然后切换终端开启gdb调试程序，输入 `set context-output /dev/pts/3`将寄存器堆栈等信息输出到另一个终端里。gdb具体用法见其他笔记
   2. pycharm联合gdb调试

      ```
      io = gdb.debug(path,Instruction)
      开启一个gdb窗口，可以与python控制台交互，在gdb窗口中调用输入输出函数时，在python控制台中发出即可被gdb进程接受

      gdb.attach(io)
      开启一个gdb窗口，运行完main函数后停止在系统调用处，与python的交互已经结束，用来查看内存中的变化,通常用于堆利用中
      ```

#### 基础知识

1. elf文件的保护机制
2. 内存空间布局
3. 寄存器和栈
4. 函数调用和传参

#### 入门题目

1. ret2text
