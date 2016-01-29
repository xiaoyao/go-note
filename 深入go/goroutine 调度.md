# goroutine 调度

#[1.调度器相关数据结构](https://tiancaiamao.gitbooks.io/go-internals/content/zh/05.1.html)


Go的调度的实现，涉及结构体G，结构体M，结构体P，以及Sched结构体

运行时库用这几个数据结构来实现goroutine的调度，管理goroutine和物理线程的运行。

## 结构体 G
G是goroutine的缩写，相当于操作系统中的进程控制块，在这里就是goroutine的控制结构，是对goroutine的抽象。其中包括goid是这个goroutine的ID，status是这个goroutine的状态。

goroutine切换时的上下文信息是保存在结构体的sched域中的。goroutine是轻量级的“线程”或者称为协程，切换时变不必陷入到操作系统内核中，所以保存过程很轻量。看一下结构体G中的Gobuf，其实只保存了当前栈指针，程序计数器，以及goroutine自身。


## 结构体M

M是machine的缩写，是对机器的抽象，每个m都是对应到一条操作系统的物理线程。M必须关联了P才可以执行Go代码，但是当它处理阻塞或者系统调用中时，可以不需要关联P。

M中还有一个MCache，是当前M的内存的缓存。M也是和G一样有一个常驻寄存器变量，代表当前的M。同时存在很多的M，就是有很多的物理线程。

结构体M中有两个G是需要关注一下的，一个是curg，代表结构体M当前绑定的结构体G。另一个是g0，是带有调度栈的goroutine，这是一个比较特殊的goroutine。普通的goroutine的栈是在堆上分配的可增长的栈，而g0的栈是M对应的线程的栈。所有调度相关的代码，会先切换到该goroutine的栈中再执行。

## 结构体P

Go1.1中新加入的一个数据结构，它是Processor的缩写。结构体P的加入是为了提高Go程序的并发度，实现更好的调度。


- **M代表OS线程。**
- **P代表Go代码执行时需要的资源。**



当M执行Go代码时，它需要关联一个P，当M为idle或者在系统调用中时，它也需要P。

- 有刚好GOMAXPROCS个P。
- 所有的P被组织为一个数组，在P上实现了工作流窃取的调度器。

注意，跟G不同的是，P的状态中是不存在waiting的。MCache是被移到了P中，但是在结构体M中也还保留着。在P中有一个Grunnable的goroutine的队列，这是一个P的局部队列。当P去执行Go代码时，它会优先从自己的这个局部队列中去取，这时可以不用加锁，提高了并发度。如果发现这个队列为空了，则去其它P中的队列中拿一半过来，这样实现工作流窃取的调度。这种情况下是需要给调用器进行加锁的。

## Sched

Sched是调度实现中使用的数据结构，该结构体的定义在文件proc.c中。

大多数需要的的信息都已放在了结构体M，结构体G和结构体P中，Sched结构体只是一个壳。可以看到，其中有M的idle队列，P的idle队列，以及一个全局的就绪的G队列。Sched结构体中的Lock是非常必须的，如果M或P等做一些非局部的操作，它们一般需要先锁住调度器。


***
# 2.goroutine的生老病死

这里主要将注意力集中到结构体G中，以goroutine为主线。


## goroutine的创建
>runtime.newproc:

一个goroutine的出生，所有的新的goroutine都是通过这个函数创建的。

runtime.newproc(size, f, args)功能就是创建一个新的g，这个函数不能用分段栈，因为它假设参数的放置顺序是紧接着函数f的.

分段栈会破坏这个布局，所以在代码中加入了标记#pragma textflag 7表示不使用分段栈。它会调用函数newproc1，在newproc1中可以使用分段栈。真正的工作是调用newproc1完成的。

>runtime.newm

功能跟newproc相似,前者分配一个goroutine,而后者分配一个M。其实一个M就是一个操作系统线程的抽象，可以看到它会调用runtime.newosproc。

## 进出系统调用
...

## goroutine的消亡以及状态变化
...
> 
在newproc1中新建的goroutine被设置为Grunnable状态，投入运行时设置成Grunning。在entersyscall的时候goroutine的状态被设置为Gsyscall，到出系统调用时根据它是从阻塞系统调用中出来还是非阻塞系统调用中出来，又会被设置成Grunning或者Grunnable的状态。在goroutine最终退出的runtime.exit函数中，goroutine被设置为Gdead状态。

![](goroutine状态变迁图.jpg)