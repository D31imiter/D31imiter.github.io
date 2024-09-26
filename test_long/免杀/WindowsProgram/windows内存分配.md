#### SYSTEM_INFO

```
typedef struct _SYSTEM_INFO {
      union {
      DWORD dwOemId;                          // 兼容性保留
      struct {
        WORD wProcessorArchitecture;          // 操作系统处理器体系结构
        WORD wReserved;                       // 保留
      } DUMMYSTRUCTNAME;
      } DUMMYUNIONNAME;
      DWORD     dwPageSize;                   // 页面大小和页面保护和承诺的粒度
      LPVOID    lpMinimumApplicationAddress;  // 指向应用程序和dll可访问的最低内存地址的指针
      LPVOID    lpMaximumApplicationAddress;  // 指向应用程序和dll可访问的最高内存地址的指针
      DWORD_PTR dwActiveProcessorMask;        // 处理器掩码
      DWORD     dwNumberOfProcessors;         // 当前组中逻辑处理器的数量
      DWORD     dwProcessorType;              // 处理器类型，兼容性保留
      DWORD     dwAllocationGranularity;      // 虚拟内存的起始地址的粒度
      WORD      wProcessorLevel;              // 处理器级别
      WORD      wProcessorRevision;           // 处理器修订
    } SYSTEM_INFO, *LPSYSTEM_INFO;
```

通过此结构体中的两个成员变量得知

`dwPageSize`：页面保护和承诺的页面大小和粒度。 这是 [VirtualAlloc](https://learn.microsoft.com/zh-cn/windows/desktop/api/memoryapi/nf-memoryapi-virtualalloc) 函数使用的页大小。

`dwAllocationGranularity`：可以分配虚拟内存的起始地址的粒度，在windows种分配内存时为固定的内存块大小，默认为64kb，0x10000

```
多此分配小块内存空间分步加载shellcode时，VirtualAlloc分配内存失败的问题
我原本使用VirtualAlloc分配0x1000字节，并且在此结束地址继续分配0x1000字节来完全加载shellcode，但是报错
是由于windows每次分配64kb块大小，VirtualAlloc微软文档说明：lpAddress参数，地址向下舍入到分配粒度中最近的倍数
因此分块加载每次最小为64kb
```

#### MEMORY_BASIC_INFORMATION

```
typedef struct _MEMORY_BASIC_INFORMATION {
  PVOID  BaseAddress;
  PVOID  AllocationBase;
  DWORD  AllocationProtect;
  SIZE_T RegionSize;
  DWORD  State;
  DWORD  Protect;
  DWORD  Type;
} MEMORY_BASIC_INFORMATION, *PMEMORY_BASIC_INFORMATION;
```

* `BaseAddress`：指定内存区域的起始地址。
* `AllocationBase`：指定内存区域的基地址。
* `AllocationProtect`：指定内存区域的保护方式。
* `RegionSize`：指定内存区域的大小。
* `State`：指定内存区域的状态（如 MEM_COMMIT、MEM_RESERVE 等）。
  ```
  MEM_COMMIT：表示内存区域已经被分配并且可用。
  MEM_RESERVE：表示内存区域已经被保留但尚未分配物理内存。
  MEM_FREE：表示内存区域是空闲的，未被分配。
  ```
* `Protect`：指定内存区域的保护方式（如 PAGE_READONLY、PAGE_READWRITE 等）。
* `Type`：指定内存区域的类型（如 MEM_PRIVATE、MEM_IMAGE 等）。
