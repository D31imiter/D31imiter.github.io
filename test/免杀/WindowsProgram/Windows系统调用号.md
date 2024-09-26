在 Windows 操作系统中，`ntoskrnl.exe` 和 `win32k.sys` 是两个核心的系统文件，分别对应于不同的功能和层次：

1. `ntoskrnl.exe`：`ntoskrnl` 是 Windows NT 内核的主要组件，它包含了操作系统的核心功能，如进程管理、内存管理、设备驱动、系统调用等。`ntoskrnl` 是操作系统的核心部分，负责处理系统级的任务和提供系统服务。在 `ntoskrnl` 中，包含了很多系统调用的实现，如进程管理、线程管理、文件系统等。`ntoskrnl` 文件通常位于 `C:\Windows\System32` 目录下。
2. `win32k.sys`：`win32k` 是 Windows 图形子系统的核心组件，它负责处理图形界面相关的任务，如窗口管理、消息处理、绘图操作等。`win32k` 主要用于支持用户界面的显示和交互，包括窗口管理、图形绘制、用户输入等功能。`win32k` 文件通常位于 `C:\Windows\System32\drivers` 目录下。

windows系统调用号表项目地址：[j00ru/windows-syscalls: Windows System Call Tables (NT/2000/XP/2003/Vista/7/8/10/11) (github.com)](https://github.com/j00ru/windows-syscalls)
