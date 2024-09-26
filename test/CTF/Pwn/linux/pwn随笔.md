#### 堆栈平衡

在system函数执行的的时候会遇到xmm寄存器相关指令，使得rsp必须按照0x10对齐

`system('/bin/sh')`获取shell条件：

1. rsp按照0x10对齐
2. 环境变量正确

#### 更换libc

Ubuntu失败kali可以：在Kali Linux上更换libc可能相对简单，因为它是一个专门用于渗透测试和安全领域的操作系统，可能允许用户更灵活地更改系统组件。而在Ubuntu系统上，更换libc可能会更加复杂，因为Ubuntu是一个通用目的的操作系统，更换系统组件可能会影响到其他软件的正常运行。此外，Ubuntu可能使用特定版本的libc与其他软件进行兼容性测试，并且有相关的依赖关系。

#### pwntools联合gdb调试

更换libc和ld时需要使用绝对路径，否则当在工作目录运行python文件时，会找不到libc和ld文件

#### ret2csu

`call    ds:(__frame_dummy_init_array_entry - 600E10h)[r12+rbx*8]`

rbx为0时，r12寄存器需要被设为对应函数got表项所在地址，就是跳转到r12处的值的地方

且这个csu只能控制rdi的低32位

#### 机器码截断

在_libc_csu_init这个函数中，一段汇编 `pop r15;ret;`对应opcode: `41 5F`，而 `5F` 恰好是 `pop rdi`的opcode

当一个函数调用为单个参数时，这就可以直接跳转到5F所在字节处，rop链：`pop rdi;ret;`

同理，例如 `41 5E 41 5F C3`，原本为 `pop r14;pop r15;ret`，从 `5e`处截断为 `pop rsi;pop r15;ret`

#### PIE程序通过堆利用获取libc基址

由于开启PIE，arena的加载地址每次会发生变化，获取libc基址需要找到libc中全局变量的偏移（全局变量是在编译时确定的，相对位置不变化），例如通过获取malloc_hook的偏移：main_arena 和 malloc_hook 物理位置上在同一页，并且靠的很近，因此，它们的地址只有后三位不一样，取malloc_hook在libc中偏移的后三位，再取程序中arena的高13位，得到程序中malloc_hook的地址，得到libc基址

#### stack migration

栈迁移时尽量在上方预留大一点的栈空间，bss段上方的空间是不可写的，调用库函数时可能向上分配栈空间到不可写的内存会造成段错误

#### read

写入利用时，不要让写入的内存块超过栈顶。在调用read函数中会调用其他函数，会将这些函数的返回地址压栈，而read写入时会覆盖这些函数的返回地址，由于是libc库中的函数，我们一般不易将这些返回地址保持原样
