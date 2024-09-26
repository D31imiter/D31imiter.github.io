### DOS部首

#### DOS'MZ'HEADER

40字节，DOS头的作用是兼容 MS-DOS 操作系统中的可执行文件

```c
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

1. **e_magic** ：一个 WORD 类型，值是一个常数 0x4D5A，用文本编辑器查看该值位‘MZ’，可执行文件必须都是'MZ'开头
2. **e_lfanew** ：为 32 位可执行文件扩展的域，用来表示 DOS头之后的NT头相对文件起始地址的偏移

#### DOS stub

大小不确定，DOS头尾部至PE文件头开始的位置是DOS stub块，其内容可以任意修改，不重要

### PE文件头

#### "PE",0,0

4字节，PE签名，固定为50 45 00 00

#### IMAGE_FILE_HEADER

20字节，标准PE头，映像文件头

```c
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;			//该文件的运行平台，对照表如下
    WORD    NumberOfSections;		//该PE文件中有多少个节，也就是节表中的项数
    DWORD   TimeDateStamp;		//PE文件的创建时间，一般有连接器填写
    DWORD   PointerToSymbolTable;	//COFF文件符号表在文件中的偏移
    DWORD   NumberOfSymbols;		//符号表的数量
    WORD    SizeOfOptionalHeader;	//紧随其后的可选映像头的大小
    WORD    Characteristics;		//可执行文件的属性，判断依据如下
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

Machine对照表：

```c
#define IMAGE_FILE_MACHINE_UNKNOWN           0
#define IMAGE_FILE_MACHINE_I386              0x014c  // Intel 386.
#define IMAGE_FILE_MACHINE_R3000             0x0162  // MIPS little-endian, 0x160 big-endian
#define IMAGE_FILE_MACHINE_R4000             0x0166  // MIPS little-endian
#define IMAGE_FILE_MACHINE_R10000            0x0168  // MIPS little-endian
#define IMAGE_FILE_MACHINE_WCEMIPSV2         0x0169  // MIPS little-endian WCE v2
#define IMAGE_FILE_MACHINE_ALPHA             0x0184  // Alpha_AXP
#define IMAGE_FILE_MACHINE_SH3               0x01a2  // SH3 little-endian
#define IMAGE_FILE_MACHINE_SH3DSP            0x01a3
#define IMAGE_FILE_MACHINE_SH3E              0x01a4  // SH3E little-endian
#define IMAGE_FILE_MACHINE_SH4               0x01a6  // SH4 little-endian
#define IMAGE_FILE_MACHINE_SH5               0x01a8  // SH5
#define IMAGE_FILE_MACHINE_ARM               0x01c0  // ARM Little-Endian
#define IMAGE_FILE_MACHINE_THUMB             0x01c2
#define IMAGE_FILE_MACHINE_AM33              0x01d3
#define IMAGE_FILE_MACHINE_POWERPC           0x01F0  // IBM PowerPC Little-Endian
#define IMAGE_FILE_MACHINE_POWERPCFP         0x01f1
#define IMAGE_FILE_MACHINE_IA64              0x0200  // Intel 64
#define IMAGE_FILE_MACHINE_MIPS16            0x0266  // MIPS
#define IMAGE_FILE_MACHINE_ALPHA64           0x0284  // ALPHA64
#define IMAGE_FILE_MACHINE_MIPSFPU           0x0366  // MIPS
#define IMAGE_FILE_MACHINE_MIPSFPU16         0x0466  // MIPS
#define IMAGE_FILE_MACHINE_AXP64             IMAGE_FILE_MACHINE_ALPHA64
#define IMAGE_FILE_MACHINE_TRICORE           0x0520  // Infineon
#define IMAGE_FILE_MACHINE_CEF               0x0CEF
#define IMAGE_FILE_MACHINE_EBC               0x0EBC  // EFI Byte Code
#define IMAGE_FILE_MACHINE_AMD64             0x8664  // AMD64 (K8)
#define IMAGE_FILE_MACHINE_M32R              0x9041  // M32R little-endian
#define IMAGE_FILE_MACHINE_CEE               0xC0EE
```

可执行文件属性的判断：将Characteristics转为2进制，若数据位为1，则具有此属性

```c
#define IMAGE_FILE_RELOCS_STRIPPED           0x0001  // Relocation info stripped from file.
#define IMAGE_FILE_EXECUTABLE_IMAGE          0x0002  // File is executable  (i.e. no unresolved externel references).
#define IMAGE_FILE_LINE_NUMS_STRIPPED        0x0004  // Line nunbers stripped from file.
#define IMAGE_FILE_LOCAL_SYMS_STRIPPED       0x0008  // Local symbols stripped from file.
#define IMAGE_FILE_AGGRESIVE_WS_TRIM         0x0010  // Agressively trim working set
#define IMAGE_FILE_LARGE_ADDRESS_AWARE       0x0020  // App can handle >2gb addresses
#define IMAGE_FILE_BYTES_REVERSED_LO         0x0080  // Bytes of machine word are reversed.
#define IMAGE_FILE_32BIT_MACHINE             0x0100  // 32 bit word machine.
#define IMAGE_FILE_DEBUG_STRIPPED            0x0200  // Debugging info stripped from file in .DBG file
#define IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP   0x0400  // If Image is on removable media, copy and run from the swap file.
#define IMAGE_FILE_NET_RUN_FROM_SWAP         0x0800  // If Image is on Net, copy and run from the swap file.
#define IMAGE_FILE_SYSTEM                    0x1000  // System File.
#define IMAGE_FILE_DLL                       0x2000  // File is a DLL.
#define IMAGE_FILE_UP_SYSTEM_ONLY            0x4000  // File should only be run on a UP machine
#define IMAGE_FILE_BYTES_REVERSED_HI         0x8000  // Bytes of machine word are reversed.
```

#### IMAGE_OPTIONAL_HEADER

扩展PE头，可选映像头

##### IMAGE_OPTIONAL_HEADER32

224字节

```c
typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD    Magic;			//表示可选头的类型,对照表如下
    BYTE    MajorLinkerVersion;		//链接器的版本号
    BYTE    MinorLinkerVersion;		//链接器的版本号
    DWORD   SizeOfCode;			//代码段的长度，如果有多个代码段，则是代码段长度的总和
    DWORD   SizeOfInitializedData;	//初始化的数据长度
    DWORD   SizeOfUninitializedData;	//未初始化的数据长度
    DWORD   AddressOfEntryPoint;	//程序入口点的相对虚拟地址
    DWORD   BaseOfCode;			//代码段起始地址的RVA
    DWORD   BaseOfData;			//代码段起始地址的RVA
    DWORD   ImageBase;			//加载到内存中的PE文件的基地址，基地址可变，对于DLL来说，如果无法加载此地址，系统会自动选择地址
    DWORD   SectionAlignment;		//节对齐，PE中的节被加载到内存时会按照此值来对齐，若值为0x1000，则每个节的起始地址的低12位都为0。
    DWORD   FileAlignment;		//节在文件中按此值对齐，SectionAlignment必须大于或等于FileAlignment
    WORD    MajorOperatingSystemVersion;//所需操作系统的版本号，随着操作系统版本数量增加，已经不重要了
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;		//映象的版本号，这个是开发者自己指定的，由连接器填写
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;	//所需子系统版本号
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;		//保留，必须为0
    DWORD   SizeOfImage;		//映象的大小，PE文件加载到内存中空间是连续的，这个值指定占用虚拟空间的大小
    DWORD   SizeOfHeaders;		//所有文件头（包括节表）的大小，这个值是以FileAlignment对齐的
    DWORD   CheckSum;			//映象文件的校验和
    WORD    Subsystem;			//运行该PE文件所需的子系统
    WORD    DllCharacteristics;		//DLL的文件属性，只对DLL文件有效
    DWORD   SizeOfStackReserve;		//运行时为每个线程栈保留内存的大小
    DWORD   SizeOfStackCommit;		//运行时每个线程栈初始占用内存大小
    DWORD   SizeOfHeapReserve;		//运行时为进程堆保留内存大小
    DWORD   SizeOfHeapCommit;		//运行时进程堆初始占用内存大小
    DWORD   LoaderFlags;		//保留，必须为0
    DWORD   NumberOfRvaAndSizes;	//数据目录的项数，即下面这个数组的项数
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];	//一个结构体数组，具体组成如下
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

##### IMAGE_OPTIONAL_HEADER64

240字节

```
typedef struct _IMAGE_OPTIONAL_HEADER64 {
    WORD        Magic;
    BYTE        MajorLinkerVersion;
    BYTE        MinorLinkerVersion;
    DWORD       SizeOfCode;
    DWORD       SizeOfInitializedData;
    DWORD       SizeOfUninitializedData;
    DWORD       AddressOfEntryPoint;
    DWORD       BaseOfCode;
    ULONGLONG   ImageBase;
    DWORD       SectionAlignment;
    DWORD       FileAlignment;
    WORD        MajorOperatingSystemVersion;
    WORD        MinorOperatingSystemVersion;
    WORD        MajorImageVersion;
    WORD        MinorImageVersion;
    WORD        MajorSubsystemVersion;
    WORD        MinorSubsystemVersion;
    DWORD       Win32VersionValue;
    DWORD       SizeOfImage;
    DWORD       SizeOfHeaders;
    DWORD       CheckSum;
    WORD        Subsystem;
    WORD        DllCharacteristics;
    ULONGLONG   SizeOfStackReserve;
    ULONGLONG   SizeOfStackCommit;
    ULONGLONG   SizeOfHeapReserve;
    ULONGLONG   SizeOfHeapCommit;
    DWORD       LoaderFlags;
    DWORD       NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
```

Magic对照表：

```c
#define IMAGE_NT_OPTIONAL_HDR32_MAGIC      0x10b  // 32位PE可选头
#define IMAGE_NT_OPTIONAL_HDR64_MAGIC      0x20b  // 64位PE可选头
#define IMAGE_ROM_OPTIONAL_HDR_MAGIC       0x107  
```

数据目录表，由16个8字节大小的结构体组成，数据目录的结构体组成：

```c
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;	//与直译不同的是，它是一个相对虚拟地址
    DWORD   Size;		//数据段的大小
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

_IMAGE_DATA_DIRECTORY指向的就是ImportDescriptor数组

关于地址概念的详解：

* 文件偏移地址（File Offset）:PE文件数据在硬盘中存放的地址就称为文件偏移地址，文件在磁盘上存放时相对于文件开头的偏移
* 装载基址（Image Base）：装载基址是指PE文件装入内存时的基地址，一般EXE文件的装载基址为 `0x00400000`，DLL为 `0x10000000`，但并不是绝对的，装载基址可以更改
* 虚拟内存地址（Virtual Address）：PE文件被装入内存之后的地址
* 相对虚拟地址（Revelitive Virtual Address）：在没有计算装载基址情况下的内存地址，即位未装载前的内存地址

若有，段起始虚拟偏移（VS），关系如下：

```
虚拟内存地址 = 装载基址 + 相对虚拟地址
相对虚拟地址 = 虚拟内存地址 - 装载基址
相对所在区段偏移 = 相对虚拟内存地址 - 段起始虚拟偏移
文件偏移 = 段起始文件偏移 + 相对所在区段偏移
文件偏移 = 段起始文件偏移 + 虚拟内存地址 - 装载基址 - 段起始虚拟偏移
```

例如BaseOfData=0x140000000，BaseOfCode=0x1000，AddressOfEntryPoint=0x14F0，BaseOfCode是代码段的开始处，AddressOfEntryPoint是程序入口点，在MSVC编译器中一般为mainCRTStartup

### SectionHeaders数组

SectionHeaders数组的成员为_IMAGE_SECTION_HEADER结构体

#### _IMAGE_SECTION_HEADER

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE  Name[8];                 // 段的名称，通常是一个长度为 8 的字符串
    union {
        DWORD PhysicalAddress;
        DWORD VirtualSize;         // 虚拟大小字段
    } Misc;
    DWORD VirtualAddress;          // 段的虚拟地址（在加载时的地址）
    DWORD SizeOfRawData;           // 段在文件中的实际大小
    DWORD PointerToRawData;        // 段在文件中的偏移量
    DWORD PointerToRelocations;    // 段中重定位表的偏移量（通常为 0）
    DWORD PointerToLinenumbers;    // 段中行号表的偏移量（通常为 0）
    WORD  NumberOfRelocations;     // 段中重定位条目的数量（通常为 0）
    WORD  NumberOfLinenumbers;     // 段中行号条目的数量（通常为 0）
    DWORD Characteristics;         // 段的属性标志
} IMAGE_SECTION_HEADER;

```

### Section

对应SectionHeaders中的各段

### ImportDescriptor数组

由IMAGE_IMPORT_DEECRIPTOR结构体组成

```c
struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics; 
        DWORD   OriginalFirstThunk;	// RVA 指向IMAGE_THUNK_DATA结构数组
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp; 		// 已绑定值为1，未绑定值为0
    DWORD   ForwarderChain;   
    DWORD   Name;			//RVA 指向dll的名字
    DWORD   FirstThunk;			//RVA 指向IMAGE_THUNK_DATA结构数组   
} IMAGE_IMPORT_DESCRIPTOR;
```

在dll未装载前，OriginalFirstThunk存放了指向INT（Import Name Table）的相对偏移量，FirstThunk存放了指向IAT（Import Adress Table）的相对偏移量，INT与IAT都由_IMAGE_THUNK_DATA64条目构成

_IMAGE_THUNK_DATA64：

```c
typedef union _IMAGE_THUNK_DATA64 {
    ULONGLONG ForwarderString;  // PBYTE 
    ULONGLONG Function;         // PDWORD64
    ULONGLONG Ordinal;
    ULONGLONG AddressOfData;    // PIMAGE_IMPORT_BY_NAME
} IMAGE_THUNK_DATA64;
```

之际上_IMAGE_THUNK_DATA64占8字节，被四种变量共享，遇到不同的情况此联合充当不同的变量

* **`ForwarderString`** ：当导入的函数是被转发的（即函数实际由另一个 DLL 提供），`ForwarderString` 字段指向转发字符串
* **`Function`** ：在导入地址表（IAT）中，`Function` 字段用于存储导入函数的实际地址。程序加载后，操作系统会将该字段替换为从导入 DLL 获取的实际函数地址
* **`Ordinal`** ：如果函数是通过序号（而非名称）导入的，那么 `Ordinal` 字段将包含这个序号值。通常这个值最高位设置为 1，以区分序号导入和名称导入
* **`AddressOfData`** ：在导入名称表（INT）中，该字段指向 `_IMAGE_IMPORT_BY_NAME` 结构，该结构包含了导入函数的名称和提示值（Hint）

在dll未加载前，INT和IAT中的_IMAGE_THUNK_DATA64都指向指向要绑定的函数的字符串，dll装载后，IAT指向了各函数的真实地址

IAT处于idata段中，INT处于rdata段中
