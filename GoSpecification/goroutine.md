# Goroutine

goroutine是通过Go的runtime管理的一个线程管理器。

goroutine通过go关键字实现了，其实就是一个普通的函数。

多个goroutine运行在同一个进程里面，共享内存数据，不过设计上我们要遵循：不要通过共享来通信，而要通过通信来共享。

channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变的更加的简单，而不需要显式的lock。所谓阻塞，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具

## runtime goroutine
runtime包中有几个处理goroutine的函数：

- Goexit

	退出当前执行的goroutine，但是defer函数还会继续调用
	
- Gosched

	让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行。

- NumCPU

	返回 CPU 核数量
	
- NumGoroutine

	返回正在执行和排队的任务总数
	
- GOMAXPROCS

	用来设置可以并行计算的CPU核数的最大值，并返回之前的值。