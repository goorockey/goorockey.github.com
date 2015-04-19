title: 《Windows核心编程》读书笔记3--线程同步
date: 2011-9-5
tags: [windows, programming]
categories: windows
---

## 原子操作

能调用的原子操作

    LONG InterlockedExchangeAdd(PLONG plAddend,LONG Increment);
    LONG InterlockedExchange(PLONG plTarget, LONG lValue);
    PVOID InterlockedExchangePointer(PVOID* ppvTarget, PVOID pvValue);
    PVOID InterlockedCompareExchange(PLONG plDestination, LONG lExchange, LONG lComparand);
    PVOID InterlockedCompareExchangePointer(PVOID* ppvDestination, PVOID pvExchange, PVOID pvComparand);

<!--more-->

## 以查询方式同步

    volatile BOOL g_fFinishedCalculation = FALSE;

    int WINAPI WinMain(…)
    {
        CreateThread(…, RecalcFunc, …);
        …

        //Wait for the recalculation to complete. 
        while(!g_fFinishedCalculation);
        …
    }

    DWORD WINAPI RecalcFunc(PVOID pvParam)
    {
        //Perform the recalculation. 
        …

        g_fFinishedCalculation = TRUE;

        return(0);
    }

- 查询的线程一直处于可调度状态，浪费CPU时间

- 如果WinMain的线程优先级比ReclcFunc的线程要高，则g\_fFinishedCalculation永远不会被置为TRUE。

## 关键代码段Critical\_Section

- 使用前调用InitializeCriticalSection进行初始化，使用后用DeleteCriticalSection释放资源

- 在指向同一个Critical\_Section的EnterCriticalSection和LeaveCriticalSection之间的代码，不会被多个线程同时调用

- 同一个线程多次重入EnterCriticalSection和LeaveCriticalSection之间的代码不会发生死锁。

如下面代码不会有死锁：

    int main(int argc, char **argv)
    {
        CRITICAL_SECTION cs;

        InitializeCriticalSection(&cs);
        EnterCriticalSection(&cs);
        EnterCriticalSection(&cs);

        while(1)
        {
            cout << “testing” << endl;
        }

        LeaveCriticalSection(&cs);
        LeaveCriticalSection(&cs);

        return 0;
    }

- 考虑到线程进入等待状态时，要保护现场，这是非常耗时的。这可以用InitializeCriticalSectionAndSpinCount，它让想进入已被占用的关键代码段的线程先循环判断多次，才进入等待状态。

- InitializeCriticalSectionAndSpinCount只对多个CPU起作用，单个CPU不起作用。

- SetCriticalSectionSpinCount可以改变循环判断的次数

- 关键代码段是在用户态实现同步的方法，这样比内核态同步要快，因为不用做用户态和内核态之间的往返（往返一次需要占用x 8 6平台上的大约1 0 0 0个C P U周期）。

## 内核对象同步

- 当内核对象是自动设置为有信号时，在所有等待该内核对象的线程中，只会有一个变为可调度，然后该内核对象又自动设为无信号。

- 当内核对象是手动设置为有信号时，除非手动设置该内核对象的状态，否则一直是有信号，这样所有等待该内核对象的线程都能变为可调度。

## WaitableTimer

- WaitableTimer能在规定时候或按规定的时间间隔变为有信号状态，就类似闹钟。

- SetWaitableTimer设置开始定时的时间（如果传参是负数，则是相对于这个函数被调用的时间）、定时的间隔、定时间隔到时调用的函数

- CancelWaitableTimer取消WaitableTimer的定时。

## 其他等待函数

--- | ---
MsgWaitForMultipleObjects和MsgWaitForMultipleObjectsEx | 等待多个内核对象有信号、或指定类型消息到达线程的输入队列
SingleObjectAndWait | 在一个原子操作完成设置一个内核对象为有信号，并进入等待另一个内核对象

## 各同步的内核对象的理解

- 关键代码段:

    critical section ,关键代码段之间的代码是原子操作，同一时间只能有一个线程执行该段代码，与别的同步object都是内核态的同步相比，它争取用用户态的方式进行同步，如果用户态的用户不行，才用内核态的同步，这样效率更高,花费较少

- 锁:

    mutex，只允许一个线程拥有
    semaphore，允许指定数量的线程拥有，创建此object时可以指定能拥有的最多的线程数

- 信号：

    event，不同于锁，就如它的名字是“信号”，当一个线程拥有锁的时候就会改变锁的状态以达到同步（`成功拥有mutex则使它无信号；成功拥有semaphore则使它计数减一，当计数为零，则semaphore变成无信号状态），手动设置的event的状态只有线程调用SetEvent或ResetEvent才会改变，线程则通过WaitForSingleObject等检测信号状态的函数来达到同步。
