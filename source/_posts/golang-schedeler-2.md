---
title: golang调度模型(二)
date: 2020-06-15 19:41:31
tags:
- golang
- runtime
- scheduler
- debug
categorise:
- golang
---
# 命令
> GODEBUG=schedtrace=1000,scheddetail=1 ./example

# 输出
> SCHED 0ms: gomaxprocs=4 idleprocs=3 threads=3 spinningthreads=0 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
> P0: status=1 schedtick=0 syscalltick=0 m=0 runqsize=0 gfreecnt=0
> M0: p=0 curg=1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 helpgc=0 spinning=false blocked=false lockedg=1
> G1: status=2(stack growth) m=0 lockedm=0

## G 
* status：G 的运行状态。
* m：隶属哪一个 M。
* lockedm：锁定 M,如果有锁定,会和m值一样

### G的状态
> _Gidle	        0	刚刚被分配，还没有进行初始化。
> _Grunnable        1	已经在运行队列中，还没有执行用户代码。
> _Grunning         2	不在运行队列里中，已经可以执行用户代码，此时已经分配了 M 和 P。
> _Gsyscall         3	正在执行系统调用，此时分配了 M。
> _Gwaiting         4	在运行时被阻止，没有执行用户代码，也不在运行队列中，此时它正在某处阻塞等待中。
> _Gmoribund_unused	5	尚未使用，但是在 gdb 中进行了硬编码。
> _Gdead	        6	尚未使用，这个状态可能是刚退出或是刚被初始化，此时它并没有执行用户代码，也有可能没有分配堆栈。
> _Genqueue_unused	7	尚未使用。
> _Gcopystack	    8	正在复制堆栈，并没有执行用户代码，也不在运行队列中。

## M
* p：隶属哪一个P.
* curg：当前正在使用哪个G。
* runqsize：运行队列中的G数量。
* gfreecnt：可用的G（状态为 Gdead）。
* mallocing：是否正在分配内存。
* throwing：是否抛出异常。
* preemptoff：不等于空字符串的话，保持 curg 在这个 m 上运行。
* locks: 表示该M是否被锁的状态，M被锁的状态下该M无法执行gc
* spinning: 是否是自旋线程
* blocked: 是否被阻塞
* lockedg: 锁定g在当前m上执行，而不会切换到其他m，一般cgo调用或者手动调用LockOSThread()才会有值

## P
* status：P 的运行状态。
* schedtick：P 的调度次数。
* syscalltick：P 的系统调用次数。
* m：隶属哪一个 M。
* runqsize：运行队列中的 G 数量。
* gfreecnt：可用的G（状态为 Gdead）。

### P的状态
> _Pidle	0	刚刚被分配，还没有进行进行初始化。
> _Prunning	1	当 M 与 P 绑定调用 acquirep 时，P 的状态会改变为 _Prunning。
> _Psyscall	2	正在执行系统调用。
> _Pgcstop	3	暂停运行，此时系统正在进行 GC，直至 GC 结束后才会转变到下一个状态阶段。
> _Pdead	4	废弃，不再使用。