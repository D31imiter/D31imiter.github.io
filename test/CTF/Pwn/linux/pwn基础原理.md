#### stack

0

1


```
call 之前：
rbp            0x7fffffffe220  
rsp            0x7fffffffe210 

0x7fffffffe210: 0x00000000      0x00000000      0x00000000      0x00000000
0x7fffffffe220: 0x00000001      0x00000000      0xf7df518a      0x00007fff

将call的下一条语句所在的内存地址压入栈
call 之后 ，push rbp 之前：
rbp            0x7fffffffe220  
rsp            0x7fffffffe208

0x7fffffffe208: 0x5555514d      0x00005555      0x00000000      0x00000000
0x7fffffffe218: 0x00000000      0x00000000      0x00000001      0x00000000

push rbp 之后：
rbp            0x7fffffffe220   
rsp            0x7fffffffe200

0x7fffffffe200: 0xffffe220      0x00007fff      0x5555514d      0x00005555
0x7fffffffe210: 0x00000000      0x00000000      0x00000000      0x00000000
0x7fffffffe220: 0x00000001      0x00000000      0xf7df518a      0x00007fff

mov    rbp,rsp：保存调用者的栈，使用新的栈帧
mov    eax,0x37：将返回值赋值给rax

pop rbp ：将栈顶的内容弹出给rbp，但是储存着旧的rbp的值位于现在的rbp处
之前有过一系列操作为
push rbp，将调用者函数的栈基址压入栈
rbp=rsp，新的栈基址为旧的栈的栈顶

当前栈底存放的是旧的栈的栈基址，应该将栈底存放的值赋值给rbp才对，而不是当前栈顶的值
但是调试后发现：进入被调用函数后，虽然被调函数一直在使用新的栈帧，但是rsp寄存器却并不增长，进入被调函数后的rsp就等于rbp，所以才可以恢复旧的栈

pop rbp之后：
rbp            0x7fffffffe220  
rsp            0x7fffffffe208

rbp恢复，而rsp经历过依次push，向上增长-0x8，pop后向下缩减，+0x8，rsp恢复
ret时将压入的rip 弹出 给rip

引发的问题，rsp到底何时增长，leave函数何时出现

之前我们发现在被调用函数func中，rsp并没有增长，所以我们可以直接pop rbp，但如果rsp增长了，pop rbp后，rbp得到的数据就不是发起调用之前的rbp了（无法恢复），这时我们才会执行leave
leave等价于：
mov rsp rbp
pop rbp

第一句mov rsp rbp即为将新栈销毁，测试案例rsp没有增长，编译器优化后直接执行第二条语句 pop rbp即可
```

#### stack总结

```
push op		=>	sub rsp,op所占的字节数
			mov rsp,op

pop op		=>	mov op,rsp
			add rsp,op所占的字节数

函数调用的总过程：

call	func  =>   push  rip;jmp  func
push	rbp		将旧的栈基址压入栈
mov 	rbp,rsp		新的栈基址为旧的栈顶，栈大小为零

被调函数的实现
将返回值赋给rax

mov 	rsp,rbp		新的栈顶等于新的栈基址，相当于销毁新栈
pop 	rbp		将旧的栈基址弹出给rbp，恢复栈
这两句指令合并，也称leave
执行leave指令或仅执行pop rbp取决于栈是否增长：若rsp改变则需要执行leave，rsp不改变仅执行pop rbp
ret	       =>   pop  rip
```

#### 64位&32位传参

```
参数的传递：从左到右依次入栈
具体细节：例如，
a=func(16,32,48,64,80,96,160,1600,16000);

   0x0000555555555175 <+15>:    push   0x3e80
   0x000055555555517a <+20>:    push   0x640
   0x000055555555517f <+25>:    push   0xa0
   0x0000555555555184 <+30>:    mov    r9d,0x60
   0x000055555555518a <+36>:    mov    r8d,0x50
   0x0000555555555190 <+42>:    mov    ecx,0x40
   0x0000555555555195 <+47>:    mov    edx,0x30
   0x000055555555519a <+52>:    mov    esi,0x20
   0x000055555555519f <+57>:    mov    edi,0x10
=> 0x00005555555551a4 <+62>:    call   0x555555555129 <func>


在调用函数之前，参数从右至左依次入栈和寄存器，（参数小于六个时，参数传递顺序为，从右至左，依次赋予，r9d，r8d，ecx，edx，esi，edi；参数大于六个时，从右至左先将多余的参数压入栈，剩余六个参数时，然后再依次赋予上述寄存器）

查看内存时，由于内存对齐，一个int占了八个字节，如下：

0x7fffffffe1e8: 0x000000a0      0x00000000      0x00000640      0x00000000
0x7fffffffe1f8: 0x00003e80      0x00000000      0x00000000      0x00000000
0x7fffffffe208: 0x00000000      0x00000000      0x00000001      0x00000000

新栈中发现了如下规律：
   0x000055555555512d <+4>:     mov    DWORD PTR [rbp-0x14],edi
   0x0000555555555130 <+7>:     mov    DWORD PTR [rbp-0x18],esi
   0x0000555555555133 <+10>:    mov    DWORD PTR [rbp-0x1c],edx
   0x0000555555555136 <+13>:    mov    DWORD PTR [rbp-0x20],ecx
   0x0000555555555139 <+16>:    mov    DWORD PTR [rbp-0x24],r8d
   0x000055555555513d <+20>:    mov    DWORD PTR [rbp-0x28],r9d
   0x0000555555555141 <+24>:    mov    DWORD PTR [rbp-0x4],0x63
   0x0000555555555148 <+31>:    mov    DWORD PTR [rbp-0x8],0xa0
   0x000055555555514f <+38>:    mov    edx,DWORD PTR [rbp+0x20]
   0x0000555555555152 <+41>:    mov    eax,DWORD PTR [rbp+0x18]
   0x0000555555555155 <+44>:    add    edx,eax
   0x0000555555555157 <+46>:    mov    eax,DWORD PTR [rbp+0x10]
   0x000055555555515a <+49>:    add    eax,edx
   0x000055555555515c <+51>:    mov    DWORD PTR [rbp-0xc],eax

用来传递参数的寄存器在新栈中为了被释放，将值依次压入栈中，后进入寄存器的值现被压入栈中
调用由栈传递的参数时，直接在旧栈中使用，通过rbp定位

```
