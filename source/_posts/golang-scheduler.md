---
title: golang调度模型(一)
date: 2020-06-13 23:32:04
tags:
- golang
- runtime
- scheduler
- debug
categorise:
- golang
---
# 基础
1. golang的调度是自己实现的,非依靠操作系统的抢占式模型.  
是协作式调度,只有当正在运行的G被阻塞或者运行结束时,别的G才会被调度.
2. runtime.NumCPU是最大的并行数.
3. 只有当正在运行的Goroutine被阻塞或者运行结束时,别的Goroutine才会被调度.常见的阻塞有:
> 阻塞的系统调用方式，比如文件或网络操作
> 垃圾自动回收  
***
# 基本组成
## G
1. G用于表示Goroutine及它所包含的当前运行的栈和状态信息.
2. G中保存的信息可以执行或恢复执行。

## M
1. M是OS线程的抽象,是物理存在的.
2. M只有和P绑定之后,才可以执行G代码.
3. M本身也不会保存G的状态,在需要任务切换时,M会将堆栈状态保存回G中,根据G中的信息恢复执行.

## P
1. P记录了G和M的相关信息,P需要调度M来让M执行G的代码.
2. 在P中包含了本地可运行的Goroutine队列.
3. 当一个新的G被创建,它会被追加在相应P队列的末尾,以保证最终会被执行。
4. 当P没有可运行的Goroutine处理时,它会随机从其他P的Goroutines队列末尾取一半G用于自己消费(steal机制).
5. 当P没有可运行的Goroutine处理时,也会有一定几率执行全局队列里的G
> ```
> func main() {
>     var result int
>     processors := runtime.GOMAXPROCS(0)  
>     for i := 0; i < processors; i++ {
>         go func() {
>             for {
>                   result++
>                   //time.Sleep(time.Second)//加上这句,造成调度,就会运行结束.
>                 }
>         }()
>     }
>     time.Sleep(time.Second)       //wait for go function to increment the value.
>     fmt.Println("result =", result)
> }
> ```
> 这段代码最后的打印结果永远得不到输出
> 因为最大的并发数为GOMAXPROCS,每个goroutine都占用一个M,每个M又都占用一个P,
> 这段代码里的G又不会阻塞,所以调度器不会运行其它的goroutine,这就造成主goroutine永远不会被继续运行下去.　
***
# 自旋线程
1. M进入了一种循环查找可运行G的状态.并不是所有的没有G的M都进入自旋,
2. 当正在自旋的M个数的2倍>=正在忙的p的个数时，不让该M进入自旋状态.
3. 自旋需要消耗CPU的,如果已经有不少的P处于忙时,应该尽量让P去占用CPU资源,而不是为了查找G而浪费CPU.
4. 如果一个M找不到G,又没有进入自旋状态,那么会休眠该M,并且会将P和M解绑.
5. 新建一个goroutine或者有个goroutine准备好时,会唤醒M或者新建M.  
***
# 阻塞
## 非channel通信造成
1. 当M准备执行Goroutine时,首选需要关联一个P,然后从P的队列中取出一个G来执行.
2. 如果G中执行的代码使M发生阻塞,那么M将会一直阻塞,直到系统调用返回,阻塞状态的M会与P解绑.
3. 此时全局空闲M队列的另一个M会被唤醒,这样做也是为了保证其他G不会因为缺少M而被阻塞执行.
4. 当阻塞的调用返回时,先前阻塞的M会尝试去偷其它线程的P来继续执行,如果没有取得,就将G放到全局的G队列中.之后进入休眠.

## channel通信造成
1. G的状态会被设置为等待,M会继续执行别的Goroutine.
2. 当G重新变成可运行状态时,等待别的M去执行.