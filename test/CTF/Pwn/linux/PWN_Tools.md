#### ELF

`elf = ELF('filepath')`获取elf文件对象

`elf.plt`获取到plt表的字典，索引为函数名，值为所在程序未加载时该plt表项相对文件的偏移

`elf.got`获取到got表的字典，索引为函数名，值为所在程序未加载时该got表项相对文件的偏移

#### LibcSearcher

`libc = LibcSearcher(func_name,func_addr)`通过LibcSearcher获取libc文件对象。知道某个函数got表项的值即可知道libc的版本,func_addr应为十进制整数

`libc_base = func_addr - libc.dump(func)`函数在elf文件中的偏移减去函数相对于函数在libc文件中偏移即可获取libc基址，因为函数在elf文件中的偏移为libc装载基址+函数在未装载的libc中的偏移

#### gdb & pwntools

`io = gdb.debug([path],[Instruction])`

开启一个gdb窗口，可以与python控制台交互，在gdb窗口中调用输入输出函数时，在python控制台中发出即可被gdb进程接受

`gdb.attach(io)`

开启一个gdb窗口，运行完main函数后停止在系统调用处，与python的交互已经结束，用来查看内存中的变化

#### ROPgadgets

`ROPgadget --binary [filename] --only "pop|ret"`

`ROPgadget --binary [filename] --sting "cat flag"`

#### Patchelf

`patchelf --set-interpreter [ld_name] [filename]`设置链接器

`patchelf --replace-needed libc.so.6 [libc_name] [filename]`设置libc

#### pwntools搜索字符串

`next(libc.search('/bin/sh'))`
