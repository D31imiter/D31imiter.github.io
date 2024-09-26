1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec level1  
   [*] '/home/kali/Desktop/level1'
       Arch:     i386-32-little
       RELRO:    Partial RELRO
       Stack:    No canary found
       NX:       NX disabled
       PIE:      No PIE (0x8048000)
       RWX:      Has RWX segments
   ```

   有可写可执行区段，栈不可执行保护没有开启
2. IDA静态分析

   ```c
   ssize_t vulnerable_function()
   {
     char buf[136]; // [esp+0h] [ebp-88h] BYREF

     printf("What's this:%p?\n", buf);
     return read(0, buf, 0x100u);
   }
   ```

   将shellcode写到buf处，再通过覆盖retn地址跳转到此处执行shellcode
