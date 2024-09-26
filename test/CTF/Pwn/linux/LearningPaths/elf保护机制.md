#### Protection

1. RELRO（ReLocation Read-Only）：分两种情况

   * Partial RELRO：got表可写
   * Full RELRO：got表只读不可写

   gcc默认开启

   ```
   -z norelro // 关闭
   -z lazy // 部分开启
   -z now // 全部开启
   ```
2. Canary：在栈帧中插入一个随机数，在函数执行完返回之前检查随机数是否被修改来判断栈是否溢出，金丝雀，堆栈溢出哨兵，栈保护技术
   gcc默认不开启

   ```
   -fno-stack-protector // 禁用
   -fstack-protector // 开启
   -fstack-protector-all // 完全开启
   ```
3. NX（no execute）：栈上的数据不可以当作代码执行，堆栈禁止执行
   gcc默认开启

   ```
   -z execstack // 禁用NX保护
   -z noexecstack // 开启NX保护
   ```
4. PIE（Position Independent Executable）：地址无关可执行文件，也称ASLR(Address Space Layout Randomization)，地址空间布局随机化，随机放置进程关键数据区域的地址空间来防止攻击者能可靠地跳转到内存的特定位置来利用函数
   gcc默认开启

   ```
   -no-pie    // 关闭PIE，0
   -fpie -pie // 开启PIE，此时强度为1：半随机 code&data、stack、mmap、vdso随机化
   -fPIE -pie // 开启PIE，此时为最高强度2：全随机 在1的基础上加上heap随机化
   ```
5. FORTIFY：轻微的检查，用于检查是否存在缓冲区溢出的错误
   gcc默认不开启

   ```
   -D_FORTIFY_SOURCE=1	// 较弱的检查
   -D_FORTIFY_SOURCE=2	// 较强的检查
   ```
6. RPATH/RUNPATH

   可以在编译时指定程序运行时动态链接库的搜寻路径，防止一些动态库被恶意替换

   ```
   gcc –Wl,-rpath              // 指定运行时动态链接库的搜寻路径，硬编码进ELF文件 “RPATH”选项。
   LD_RUN_PATH 环境变量         // 指定运行时动态链接库的搜寻路径，硬编码进ELF文件 “RPATH”选项。

   -Wl,--disable-new-dtags     // 表明使用 RPATH 
   -Wl,--enable-new-dtags      // 标示使用 RUNPATH 
   ```
