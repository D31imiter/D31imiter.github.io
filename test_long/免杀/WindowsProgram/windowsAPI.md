# WindowsAPI

#### 用户态API

##### VirtualAlloc

```
LPVOID VirtualAlloc(  
  LPVOID lpAddress,        // region to reserve or commit
  SIZE_T dwSize,           // size of region
  DWORD flAllocationType,  // type of allocation
  DWORD flProtect          // type of access protection
);
```

* lpAddress：分配位置，为NULL时系统分配
* dwSize：内存大小
* flAllocationType：分配手段

  ```
  MEM_COMMIT: 分配物理内存并映射到进程的虚拟地址空间。
  MEM_RESERVE: 预留一块虚拟地址空间，但不分配物理内存。
  MEM_RESET: 将指定区域的内容重置为零，并且使该区域的内容失效。
  MEM_LARGE_PAGES: 使用大页面进行分配。
  MEM_PHYSICAL: 分配物理内存。
  ```
* flProtect：内存区域的保护属性

  ```
  PAGE_READONLY: 只读访问。
  PAGE_READWRITE: 读写访问。
  PAGE_EXECUTE: 仅可执行。
  PAGE_EXECUTE_READ: 可执行和只读访问。
  PAGE_EXECUTE_READWRITE: 可执行和读写访问。
  ```

##### CreateThread

```
HANDLE CreateThread(  
  LPSECURITY_ATTRIBUTES lpThreadAttributes, // SD
  DWORD dwStackSize,                        // initial stack size
  LPTHREAD_START_ROUTINE lpStartAddress,    // thread function
  LPVOID lpParameter,                       // thread argument
  DWORD dwCreationFlags,                    // creation option
  LPDWORD lpThreadId                        // thread identifier
);
```

* `lpThreadAttributes`：指定线程的安全属性，通常传入 NULL。
* `dwStackSize`：指定线程栈的大小，通常传入 0，表示使用默认大小。
* `lpStartAddress`：指定线程函数的起始地址，即新线程将要执行的函数。
* `lpParameter`：传递给线程函数的参数，可以是任意类型的指针。
* `dwCreationFlags`：指定线程的创建标志，如优先级、堆栈大小等。
* `lpThreadId`：用于接收新线程的标识符。

##### InternetIOpen

```
INTERNETAPI_(HINTERNET) InternetOpenW(
    _In_opt_ LPCWSTR lpszAgent,
    _In_ DWORD dwAccessType,
    _In_opt_ LPCWSTR lpszProxy,
    _In_opt_ LPCWSTR lpszProxyBypass,
    _In_ DWORD dwFlags
);
```

* `lpszAgent`：指定一个标识应用程序的字符串，通常为应用程序的名称。
* `dwAccessType`：指定应用程序的访问类型，可以是 `INTERNET_OPEN_TYPE_DIRECT`（直接连接）、`INTERNET_OPEN_TYPE_PROXY`（通过代理服务器连接）等。
* `lpszProxyName`：指定代理服务器的名称。
* `lpszProxyBypass`：指定不需要代理的地址。
* `dwFlags`：指定一些标志位，如 `INTERNET_FLAG_ASYNC`（异步操作）、`INTERNET_FLAG_NO_CACHE_WRITE`（禁止缓存写入）等。

##### InternetConnect

```
HINTERNET InternetConnect(
  HINTERNET hInternet,
  LPCTSTR   lpszServerName,
  INTERNET_PORT nServerPort,
  LPCTSTR   lpszUsername,
  LPCTSTR   lpszPassword,
  DWORD     dwService,
  DWORD     dwFlags,
  DWORD_PTR dwContext
);
```

* `hInternet`：调用 `InternetOpen` 函数时返回的 Internet 会话句柄。
* `lpszServerName`：指定目标服务器的主机名或 IP 地址。
* `nServerPort`：指定服务器端口号。
* `lpszUsername` 和 `lpszPassword`：指定连接服务器时使用的用户名和密码（可选）。
* `dwService`：指定服务类型，如 `INTERNET_SERVICE_FTP`、`INTERNET_SERVICE_HTTP`。
* `dwFlags`：指定连接的标志位，如 `INTERNET_FLAG_PASSIVE`、`INTERNET_FLAG_SECURE`。
* `dwContext`：指定一个应用程序定义的上下文参数，可用于回调函数中。

##### HttpOpenRequset

```
HINTERNET HttpOpenRequest(
  HINTERNET hConnect,
  LPCTSTR   lpszVerb,
  LPCTSTR   lpszObjectName,
  LPCTSTR   lpszVersion,
  LPCTSTR   lpszReferrer,
  LPCTSTR   *lplpszAcceptTypes,
  DWORD     dwFlags,
  DWORD_PTR dwContext
);
```

* `hConnect`：调用 `InternetConnect` 函数时返回的连接句柄。
* `lpszVerb`：指定 HTTP 请求的动作，如 `GET`、`POST` 等。
* `lpszObjectName`：指定请求的对象，通常是服务器上的资源路径。
* `lpszVersion`：指定 HTTP 协议的版本，如 "HTTP/1.1"。
* `lpszReferrer`：指定引用页面的 URL。
* `lplpszAcceptTypes`：指定可接受的 MIME 类型。
* `dwFlags`：指定请求的标志位，如 `INTERNET_FLAG_SECURE`。
* `dwContext`：指定一个应用程序定义的上下文参数，可用于回调函数中。

##### HttpSengRequest

```
BOOL HttpSendRequest(
  HINTERNET hRequest,
  LPCTSTR   lpszHeaders,
  DWORD     dwHeadersLength,
  LPVOID    lpOptional,
  DWORD     dwOptionalLength
);
```

* `hRequest`：调用 `HttpOpenRequest` 函数时返回的 HTTP 请求句柄。
* `lpszHeaders`：指定要发送的请求头信息。
* `dwHeadersLength`：指定请求头信息的长度。
* `lpOptional`：指定要发送的可选数据。
* `dwOptionalLength`：指定可选数据的长度。

##### InternetReadFile

```
BOOL InternetReadFile(
  HINTERNET hFile,
  LPVOID    lpBuffer,
  DWORD     dwNumberOfBytesToRead,
  LPDWORD   lpdwNumberOfBytesRead
);
```

* `hFile`：表示一个已经打开的 Internet 资源的句柄，通常是通过 `HttpOpenRequest` 和 `HttpSendRequest` 返回的请求句柄。
* `lpBuffer`：指向一个缓冲区，用于存储读取的数据。
* `dwNumberOfBytesToRead`：指定要读取的字节数。
* `lpdwNumberOfBytesRead`：指向一个变量，用于存储实际读取的字节数。

##### HeapCreate

```
HANDLE HeapCreate(
  DWORD flOptions,
  SIZE_T dwInitialSize,
  SIZE_T dwMaximumSize
);
```

* `flOptions`：指定堆的行为选项，可以是以下标志的组合：
  * `HEAP_GENERATE_EXCEPTIONS`：当堆操作失败时，生成异常。
  * `HEAP_NO_SERIALIZE`：禁用堆的序列化。
  * `HEAP_ZERO_MEMORY`：在分配的内存块中填充零。
* `dwInitialSize`：指定堆的初始大小，以字节为单位。
* `dwMaximumSize`：指定堆的最大大小，以字节为单位。如果为 0，则表示堆可以动态增长。

##### VirtualQuery

```
SIZE_T VirtualQueryEx(
  HANDLE                    hProcess,
  LPCVOID                   lpAddress,
  PMEMORY_BASIC_INFORMATION lpBuffer,
  SIZE_T                    dwLength
);
```

* hProcess：进程句柄。需要查询的进程的句柄
* lpAddress：基地址。需要查询的内存块的基地址
* lpBuffer：内存信息缓冲区。 PMEMORY_BASIC_INFORMATION 结构指针，用于存储查询结果。它包含了取得的内存块信息，如基地址、保护属性、状态、大小等
* dwLength：缓冲区大小。缓冲区的大小，以字节为单位。如果缓冲区太小，则函数将返回指定的内存块信息长度存放到此处，不会写入

##### GetProcAddress

```
FARPROC GetProcAddress(
  HMODULE hModule,
  LPCSTR  lpProcName
);
```

* `hModule`：要获取导出函数地址的 DLL 模块的句柄。
* `lpProcName`：要获取地址的函数名称。

##### WriteProcessMemory

```
BOOL WriteProcessMemory(
  HANDLE  hProcess,
  LPVOID  lpBaseAddress,
  LPCVOID lpBuffer,
  SIZE_T  nSize,
  SIZE_T  *lpNumberOfBytesWritten
);
```

* `hProcess`：目标进程的句柄，表示要向其写入数据的进程。
* `lpBaseAddress`：要写入数据的目标地址。
* `lpBuffer`：要写入的数据的缓冲区。
* `nSize`：要写入的数据大小，以字节为单位。
* `lpNumberOfBytesWritten`：一个指向 `SIZE_T` 类型的指针，用于接收成功写入的字节数。

##### EnumProcessMoudles

```
BOOL EnumProcessModules(
  HANDLE  hProcess,
  HMODULE *lphModule,
  DWORD   cb,
  LPDWORD lpcbNeeded
);
```

* `hProcess`：要获取模块句柄的进程句柄。
* `lphModule`：接收模块句柄数组的指针。
* `cb`：指定 `lphModule` 缓冲区的大小，以字节为单位。
* `lpcbNeeded`：接收所需缓冲区大小的指针。

##### ReadFile

```
BOOL ReadFile(
  HANDLE       hFile,
  LPVOID       lpBuffer,
  DWORD        nNumberOfBytesToRead,
  LPDWORD      lpNumberOfBytesRead,
  LPOVERLAPPED lpOverlapped
);
```

* `hFile`：要读取数据的文件句柄。
* `lpBuffer`：指向用来存放读取数据的缓冲区的指针。
* `nNumberOfBytesToRead`：要读取的字节数。
* `lpNumberOfBytesRead`：指向一个变量，用来存放实际读取的字节数。
* `lpOverlapped`：指向一个 `OVERLAPPED` 结构体的指针，用于实现异步 I/O 操作，通常为 NULL。

##### CreateRemoteThread

```
HANDLE CreateRemoteThread(
  HANDLE                 hProcess,
  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  SIZE_T                 dwStackSize,
  LPTHREAD_START_ROUTINE lpStartAddress,
  LPVOID                 lpParameter,
  DWORD                  dwCreationFlags,
  LPDWORD                lpThreadId
);
```

* `hProcess`：目标进程的句柄。
* `lpThreadAttributes`：线程的安全属性，通常为 NULL。
* `dwStackSize`：新线程的堆栈大小，通常为 0。
* `lpStartAddress`：线程函数的地址。
* `lpParameter`：传递给线程函数的参数。
* `dwCreationFlags`：线程创建标志。
* `lpThreadId`：用于接收新线程标识符的变量。

##### CreateRemoteThreadEx

```
HANDLE CreateRemoteThreadEx(
  HANDLE                hProcess,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  SIZE_T                dwStackSize,
  LPTHREAD_START_ROUTINE lpStartAddress,
  LPVOID                lpParameter,
  DWORD                 dwCreationFlags,
  LPPROC_THREAD_ATTRIBUTE_LIST lpAttributeList,
  LPDWORD               lpThreadId
);
```

* `hProcess`：要在其上创建线程的目标进程的句柄。
* `lpThreadAttributes`：线程的安全属性。
* `dwStackSize`：要为新线程分配的堆栈大小。
* `lpStartAddress`：新线程的入口点地址，即线程将从此地址开始执行。
* `lpParameter`：传递给新线程函数的参数。
* `dwCreationFlags`：线程的创建标志。
* `lpAttributeList`：线程属性列表。
* `lpThreadId`：返回新线程的线程ID。

##### Openprocess

```
HANDLE OpenProcess(
  DWORD dwDesiredAccess,
  BOOL  bInheritHandle,
  DWORD dwProcessId
);
```

* `dwDesiredAccess`：指定打开进程的访问权限，可以是以下常用权限的组合：
  * `PROCESS_ALL_ACCESS`：拥有完全访问权限。
  * `PROCESS_CREATE_THREAD`：允许创建线程。
  * `PROCESS_QUERY_INFORMATION`：允许查询有关进程的信息。
  * `PROCESS_VM_READ`：允许读取进程内存。
  * 等等。具体权限可以查阅相关文档。
* `bInheritHandle`：指定返回的句柄是否可以被子进程继承，通常设为 FALSE。
* `dwProcessId`：要打开进程的进程标识符（PID）。

##### CreateFileMapping

```
HANDLE CreateFileMapping(
  HANDLE                hFile,
  LPSECURITY_ATTRIBUTES lpAttributes,
  DWORD                 flProtect,
  DWORD                 dwMaximumSizeHigh,
  DWORD                 dwMaximumSizeLow,
  LPCTSTR               lpName
);
```

* `hFile`：要映射的文件句柄，如果不需要将文件映射到内存中，可以设置为 `INVALID_HANDLE_VALUE`。
* `lpAttributes`：安全属性，用于指定文件映射对象的安全描述符。
* `flProtect`：指定文件映射对象的访问权限，如读取、写入等。
* `dwMaximumSizeHigh` 和 `dwMaximumSizeLow`：指定文件映射对象的最大大小。
* `lpName`：文件映射对象的名称，用于标识文件映射对象。

##### MapViewOfMap

```
LPVOID MapViewOfFile(
  HANDLE hFileMappingObject,
  DWORD  dwDesiredAccess,
  DWORD  dwFileOffsetHigh,
  DWORD  dwFileOffsetLow,
  SIZE_T dwNumberOfBytesToMap
);
```

* `hFileMappingObject`：要映射的文件句柄或其他内核对象的句柄。
* `dwDesiredAccess`：指定映射对象的访问权限，如读取、写入等。
* `dwFileOffsetHigh` 和 `dwFileOffsetLow`：指定文件映射的起始偏移量。
* `dwNumberOfBytesToMap`：要映射的字节数。

#### 内核API

##### ZwAllocateVirtualMemory

```
NTSTATUS ZwAllocateVirtualMemory(
  HANDLE    ProcessHandle,
  PVOID     *BaseAddress,
  ULONG_PTR ZeroBits,
  PSIZE_T   RegionSize,
  ULONG     AllocationType,
  ULONG     Protect
);
```

* `ProcessHandle`：要分配内存的目标进程的句柄。
* `BaseAddress`：指向要分配内存的地址的指针。
* `ZeroBits`：保留参数，通常设置为 0。
* `RegionSize`：要分配的内存大小。
* `AllocationType`：内存分配类型，如 MEM_COMMIT、MEM_RESERVE 等。
* `Protect`：内存保护属性，如 PAGE_READWRITE、PAGE_EXECUTE_READWRITE。

##### NTCreateThread

```
NTSYSCALLAPI
NTSTATUS
NTAPI
NtCreateThread(
    _Out_ PHANDLE ThreadHandle,
    _In_ ACCESS_MASK DesiredAccess,
    _In_opt_ POBJECT_ATTRIBUTES ObjectAttributes,
    _In_ HANDLE ProcessHandle,
    _Out_ PCLIENT_ID ClientId,
    _In_ PCONTEXT ThreadContext,
    _In_ PINITIAL_TEB InitialTeb,
    _In_ BOOLEAN CreateSuspended
    );
```

* `ThreadHandle` - a pointer to a variable that receives a handle to the new thread.
* `DesiredAccess` - the thread access mask to provide on the returned handle. This value is usually `<a href="https://ntdoc.m417z.com/thread_all_access">THREAD_ALL_ACCESS</a>`.
* `ObjectAttributes` - an optional pointer to an `<a href="https://ntdoc.m417z.com/object_attributes">OBJECT_ATTRIBUTES</a>` structure that specifies attributes for the new object/handle, such as the security descriptor and handle inheritance.
* `ProcessHandle` - a handle to the process where the thread should be created. This can either be the `<a href="https://ntdoc.m417z.com/ntcurrentprocess">NtCurrentProcess</a>` pseudo-handle or a handle with `<a href="https://ntdoc.m417z.com/process_create_thread">PROCESS_CREATE_THREAD</a>` access.
* `ClientId` - a pointer to a variable that receives the client ID of the new thread.
* `ThreadContext` - the initial context (a set of registers) for the thread.
* `InitialTeb` - the structure describing the thread stack.
* `CreateSuspended` - whether the new thread should be created in a suspended state or allowed to run immediately. When specifying `TRUE`, you can use `<a href="https://ntdoc.m417z.com/ntresumethread">NtResumeThread</a>` to resume the thread later.

##### NTCreateThreadEx

`NtCreateThreadEx`是 `CreateRemoteThread`的底层函数

```
#ifdef _AMD64_
typedef DWORD(WINAPI* PfnZwCreateThreadEx)(
    PHANDLE ThreadHandle,
    ACCESS_MASK DesiredAccess,
    LPVOID ObjectAttributes,
    HANDLE ProcessHandle,
    LPTHREAD_START_ROUTINE lpStartAddress,
    LPVOID lpParameter,
    ULONG CreateThreadFlags,
    SIZE_T ZeroBits,
    SIZE_T StackSize,
    SIZE_T MaximunStackSize,
    LPVOID pUnkown);
 
 
#else
 
typedef DWORD(WINAPI *PfnZwCreateThreadEx)(
    PHANDLE ThreadHandle,
    ACCESS_MASK DesiredAccess,
    LPVOID ObjectAttributes,
    HANDLE ProcessHandle,
    LPTHREAD_START_ROUTINE lpStartAddress,
    LPVOID lpParameter,
    BOOL CreateThreadFlags,
    DWORD  ZeroBits,
    DWORD  StackSize,
    DWORD  MaximumStackSize,
    LPVOID pUnkown);
 
#endif // DEBUG
```
