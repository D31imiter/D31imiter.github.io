#### ELF Header

```
typedef struct elf32_hdr
{
	  unsigned char	e_ident[EI_NIDENT];	/* Magic number and other info */
	  Elf32_Half	e_type;			/* Object file type */
	  Elf32_Half	e_machine;		/* Architecture */
	  Elf32_Word	e_version;		/* Object file version */
	  Elf32_Addr	e_entry;		/* Entry point virtual address */
	  Elf32_Off	e_phoff;		/* Program header table file offset */
	  Elf32_Off	e_shoff;		/* Section header table file offset */
	  Elf32_Word	e_flags;		/* Processor-specific flags */
	  Elf32_Half	e_ehsize;		/* ELF header size in bytes */
	  Elf32_Half	e_phentsize;		/* Program header table entry size */
	  Elf32_Half	e_phnum;		/* Program header table entry count */
	  Elf32_Half	e_shentsize;		/* Section header table entry size */
	  Elf32_Half	e_shnum;		/* Section header table entry count */
	  Elf32_Half	e_shstrndx;		/* Section header string table index */
} Elf32_Ehdr;
```

`e_ident`：16字节，最开始处的这 16 个字节含有 ELF 文件的识别标志，作为一个数组，它的各个索引位置的字节数据有固定的含义，提供一些用于解码和解析文件内容的数据，是不依赖于具体操作系统的， `e_ident`数组结构：

```
EI_MAG0，EI_MAG1，EI_MAG2，EI_MAG3，前四字节为固定的.ELF,固定编码为7f 45 4c 46, 
EI_CLASS,1字节，表明文件类型，以下为文件类型对照表：
#define EI_CLASS	4		/* File class byte index */
#define ELFCLASSNONE	0		/* Invalid class */
#define ELFCLASS32	1		/* 32-bit objects */
#define ELFCLASS64	2		/* 64-bit objects */
#define ELFCLASSNUM	3
EI_DATA，1字节，表编码格式，以下为编码类型对照表：
#define EI_DATA		5		/* Data encoding byte index */
#define ELFDATANONE	0		/* Invalid data encoding */
#define ELFDATA2LSB	1		/* 2's complement, little endian */
#define ELFDATA2MSB	2		/* 2's complement, big endian */
#define ELFDATANUM	3
EI_VERSION,1字节，ELF文件头的版本
EI_OSABI（ELF Identification-Operating System Application Binary Interface Identification），1字节，指明 ELF 文件操作系统的二进制接口的版本标识符
EI_ABIVERSION，1字节，指明 ELF 文件的 ABI 版本
EI_PAD，8字节，最后的字节用与对其，无具体意义
```

`e_type`：2字节，文件类型

```
#define ET_NONE		0		/* No file type */
#define ET_REL		1		/* Relocatable file */
#define ET_EXEC		2		/* Executable file */
#define ET_DYN		3		/* Shared object file */
#define ET_CORE		4		/* Core file */
#define	ET_NUM		5		/* Number of defined types */
#define ET_LOOS		0xfe00		/* OS-specific range start */
#define ET_HIOS		0xfeff		/* OS-specific range end */
#define ET_LOPROC	0xff00		/* Processor-specific range start */
#define ET_HIPROC	0xffff		/* Processor-specific range end */
```

`e_machine`：2字节，用于指定该文件适用的处理器体系结构

```
#define EM_X86_64	62	         /* AMD x86-64 architecture */
```

`e_version`：4字节，指明目标文件的版本

```
/* Legal values for e_version (version).  */

#define EV_NONE		0		 /* Invalid ELF version */
#define EV_CURRENT	1		 /* Current version */
#define EV_NUM		2
```

`e_entry`：此字段（64 位 ELF 文件是 8 字节）指明程序入口的虚拟地址

`e_phoff`（ELF Header-Program Header Table Offset）：8 字节，指明程序头表（program header table）开始处在文件中的偏移量，相对于 ELF 文件初始位置的偏移量。程序头表又称为段头表，上面介绍过 ELF 的执行试图中涉及到若干的段，而程序头表包含这些段的一个总览的信息。如果没有程序头表，该值应设为 0

`e_shoff`（ELF Header-Section Header Table Offset）：8 字节，指明节头表（section header table）开始处在文件中的偏移量。如果没有节头表，该值应设为 0。`e_shoff`与之后要介绍的 `e_shentsize`和 `e_shnum`这三个成员描述了 ELF 文件中关于节头表部分的信息，`e_shoff`：起始地址偏移，节头表开始的位置；

`e_shentsize`：节头表中每个表项的大小

`e_shnum`：表项的数量

`e_flags`：4 字节，含有处理器特定的标志位。对于 Intel 架构的处理器来说，它没有定义任何标志位，所以 e_flags 应该值为 0。

`e_ehsize`：2 字节，表明 ELF 文件头的大小，以字节为单位

`e_phentsize`：2 字节，表明在程序头表中每一个表项的大小，以字节为单位。在 ELF 文件的其他数据结构中也有相同的定义方式，如果一个结构由若干相同的子结构组成，则这些子结构就称为入口。

`e_phnum`：2 字节，表明程序头表中总共有多少个表项。如果一个目标文件中没有程序头表，该值应设为 0。

`e_shentsize`：2 字节，表明在节头表中每一个表项的大小，以字节为单位。

`e_shnum`：2 字节，表明节头表中总共有多少个表项。如果一个目标文件中没有节头表，该值应设为 0。

`e_shstrndx`：2 字节，表明节头表中与节名字表相对应的表项的索引。如果文件没有节名字表，此值应设置为 SHN_UNDEF。

#### Program header table

列举了所有有效的段(segments)和他们的属性
