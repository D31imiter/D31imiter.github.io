#### winpwn保护机制

1. DEP（Data Execution Prevention）：数据执行保护，堆栈不可执行保护
2. GS（Guarded Stack）：相当于linux中的canary
3. ASLR：exe程序基址同和dll装载基址都随机化
4. Dynamic Base：仅dll装载基址随机化
5. HEVA（High Entropy Virtual Address ）：是一种增强的 ASLR 机制，通过增加内存地址的随机化位数，提高了内存地址的随机化程度
6. CFG（Control Flow Guard）：在编译时，CFG 创建一个控制流表，记录所有合法的跳转目标（例如函数地址）。在程序运行时，CFG 会验证跳转目标是否在控制流表中。如果跳转目标不在表中，程序将终止

#### windows传参

x64程序传参顺序：从右到左依次使用RCX，RDX，R8，R9，剩余参数压入栈中

#### ret2dll

同ret2libc相同，需要在ucrtbase.dll中找到system并调用，具体还是泄露dll基址，然后构造system("cmd.exe")
