title: 《Windows核心编程》读书笔记2--进程、作业、线程
date: 2011-9-3
tags: [windows, programming]
categories: windows
---

## 一、摘要：

- 进程只是线程的容器，存放数据和代码，但不执行代码。

- 线程才是执行代码的实体。

- 作业是对一个或多个进程的统一管理，能添加一般无法添加的限制。

## 二、进程

1.概念

**进程只是线程的容器**，为线程执行代码提供资源、营造运行环境。

<!--more-->

2.进程的构成

- 关键：

> 一块内存地址空间，用以存放代码和数据；
> 一个内核对象句柄表，记录使用中的内核对象。

- 更详尽的构成

(来自MSDN <http://msdn.microsoft.com/zh-cn/library/ms681917(v=vs.85).aspx>):

    a virtual address space,
    executable code,
    open handles to system objects,
    a security context,
    a unique process identifier,
    environment variables,
    a priority class,
    minimum and maximum working set sizes,
    at least one thread of execution.
 
3.进程的终止

- 全部线程都结束。即使主线程退出了，如果还有线程存在，该进程仍然不会销毁。

- ExitProcess，有可能造成内存泄露，因为C/C++ Runtime Library没有被清空，则全局变量等资源就不会被释放。

- TerminateProcess，跟ExitProcess一样是可能造成内存泄露的。另外它是异步的，即只是通知要终止目标进程，返回后并不代表它已结束。

## 三、作业

1.基本概念

- **作业是进程的容器**，对一个或多个进程附加一定的限制，进行统一管理。

- 即使作业只包含了一个进程也是有用的，因为这样能做一些普通不能进行的限制

2.主要的API:

API | 功能
--- | ---
CreateJobObject | 创建作业内核对象
OpenJobObject | 根据Handle打开作业内核对象
IsProcessInJob | 验证某一个进程是否存在于作业中
SetInformationJobObject | 给作业加上各种限制
QueryInformationJobObject | 查询作业对象的信息
AssignProcessToJobObject | 将进程放入作业
TerminateJobObject | 终止作业内所有进程

3.用于作业对象的基本用户界面限制的位标志

标志 | 描述
---- | ---
JOB\_OBJECT\_UILIMIT\_EXITWINDOWS | 用于防止进程通过ExitWindowsEx函数退出、关闭、重新引导或关闭系统电源
JOB\_OBJECT\_UILIMIT\_READCLIPBOARD | 防止进程读取剪贴板的内容
JOB\_OBJECT\_UILIMIT\_WRITECLIPBOARD | 防止进程删除剪贴板的内容
JOB\_OBJECT\_UILIMIT\_SYSTEMPARAMETERS | 防止进程通过SystemParametersInfor函数来改变系统参数
JOB\_OBJECT\_UILIMIT\_DISPLAYSETTINGS | 防止进程通过ChangeDisplaySettings函数来改变显示设置
JOB\_OBJECT\_UILIMIT\_GLOBALATOMS | 为作业赋予它自己的基本结构表，使作业中的进程只能访问该作业的表
JOB\_OBJECT\_UILIMIT\_DESKTOP | 防止进程使用CreateDesktop或SwitchDesktop函数创建或转换桌面
JOB\_OBJECT\_UILIMIT\_HANDLES | 防止作业中的进程使用同一作业外部的进程创建的USER对象（如HWND）

## 四、线程

1.基本概念

- 进程不执行代码的，**是线程在进程地址空间内执行代码**，并对进程地址空间内的数据做操作。

- 多个线程共享进程内的地址空间，包括进程的内核对象句柄表，因为句柄表的存在依赖于进程，而不是线程。

2.线程的构成

- 关键：

    一个堆栈、
    一些用于保护线程的寄存器、
    一个指令寄存器（IP）、
    堆栈指针寄存器（SP）

- 更详尽的构成

（来自MSDN <http://msdn.microsoft.com/zh-cn/library/ms681917(v=vs.85).aspx>）：

    All threads of a process share its virtual address space and system resources.
    In addition, each thread maintains exception handlers, a scheduling priority, thread local storage, a unique thread identifier, and a set of structures the system will use to save the thread context until it is scheduled.
    The thread context includes the thread's set of machine registers, the kernel stack, a thread environment block, and a user stack in the address space of the thread's process.
    Threads can also have their own security context, which can be used for impersonating clients.

3.线程的启动

- 初始化线程时会把 线程函数的入参(pvParam)、线程函数的指针(pfnStartAddrj) 压栈。

![线程堆栈](http://www.goorockey.com/uploads/2011/09/clip_image0011.png)

- 每个线程还有一个指令寄存器（IP）和堆栈指针寄存器（SP）。IP初始指向BaseThreadStart函数，它包含在Kernel32.dll中。

它主要是调用线程函数，并把函数返回值传给ExitThread：

    VOID BaseThreadStart(PTHREAD_START_ROUTINE pfnStartAddr,PVOID pvParam)
    {
        __try
        {
            ExitThread((pfnStartAddr)(pvParam));
        }
        __except(UnhandledExceptionFilter(GetExceptionInformation()))
        {
            ExitProcess(GetExceptionCode());
        }
        //NOTE: We never get here. 
    }

- 之所以pfnStartAddr和pvParam压栈，就是因为线程开始运行时，CPU跳到IP指向BaseThreadStart，然后把pfnStartAddr和pvParam出栈，就把它们当做形参传给BaseThreadStart了。

## 五、其他

1.C/C++ Runtime Library的多线程版本

- 在C/C++ Runtime Library中，有一些全局变量。它们有可能同时被多个线程访问，使它们的值无法确定。

- C/C++ Runtime Library为了适应多线程，出现多线程(MT)版本，改变一些全局变量和函数的特性。

- 主要思路是为每个线程关联一个数据结构**tiddata块**，里面都有各全局变量对于这个线程的副本。即每个线程访问的是属于自己的“全局变量”，有属于自己的独立环境。

- 而相关的函数对这些全局变量的操作也会改为对于**tiddata块**对于值的操作。

例如：

    _beginthreadex就是在调用CreateThread来创建线程的基础上，在线程初始化时创建线程关联的**tiddata块**，并把这些全局变量拷贝到里面。所以_beginthreadex比CreateThread要安全。
    _endthreadex则是对应多做了清空关联数据结构的操作。

- 如果在多线程版本的C/C++ Runtime Library中，用了CreateThread来创建线程，则线程初始化时不会有**tiddata块**。而当函数要访问**tiddata块**的时候，开始会访问失败，然后会自动生成一个，并把它与线程关联起来。但在一些情况下调用CreateThread就可能出现错误。

- \_beginthread比\_beginthreadex、以及\_endthread比\_endthreadex的参数要少，少了对线程安全访问权的控制。

2.伪句柄

- 用GetCurrentThread和GetCurrentProcess得到句柄是自己句柄的引用，并不会使线程进程的使用计数加1，它们返回的句柄叫伪句柄。

- 用CloseHandle关闭伪句柄时，会返回FALSE。

3.纤程

- UNIX服务器应用程序属于单线程程序（Windows定义），但其内部仿真了多线程工作。为了方便把UNIX服务器应用程序移植到Windows，就推出了纤程。

- ConvertThreadToFiber 把线程转换为纤程。

- 纤程不应该返回，返回会使线程和该线程所有的纤程都撤销。

- 在单个线程里，每次只能运行一个纤程。可以用SwtichToFiber来切换纤程
