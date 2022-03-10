# 协程、线程、进程



### go协程的特点

goroutine 是协程的 Go 语言实现，它是语言原生支持的，相对于一般由库实现协程的方式，goroutine 更加强大，它的调度一定程度上是由 go 运行时（runtime）管理。其好处之一是，当某 goroutine 发生阻塞时（例如同步IO操作等），会自动出让 CPU 给其它 goroutine。goroutine是非常轻量级的，它就是一段代码，一个 **[函数](https://haicoder.net/golang/golang-func.html)** 入口，以及在堆上为其分配的一个堆栈（初始大小为4K，会随着程序的执行自动增长删除）。所以它非常廉价，我们可以很轻松的创建上万个 goroutine。



### go协程的基本调度

默认的，所有 goroutine 会在一个原生线程里跑，也就是只使用了一个 CPU 核。在同一个原生线程里，如果当前goroutine  不发生阻塞，它是不会让出 CPU 时间给其他同线程的 goroutines 的。除了被系统调用阻塞的线程外，Go 运行库最多会启动  $GOMAXPROCS 个线程来运行 goroutine。

那么 goroutine 究竟是如何被调度的呢？我们从 go 程序启动开始说起。在go程序启动时会首先创建一个特殊的内核线程 sysmon，从名字就可以看出来它的职责是负责监控的，goroutine 背后的调度可以说就是靠它来搞定。

接下来，我们再看看它的调度模型，go 语言当前的实现是 N:M。即一定数量的用户线程映射到一定数量的 OS 线程上，这里的用户线程在 go 中指的就是 goroutine。go语言的调度模型需要弄清楚三个概念：M、P 和 G，如下图表示：

![调度示意图1](D:\typora\code\go\协程基本概念\协程调度示意图1.png)

M 代表 OS 线程，G 代表 goroutine，P 的概念比较重要，它表示执行的上下文，其数量由 $GOMAXPROCS  决定，一般来说正好等于处理器的数量。M 必须和 P 绑定才能执行 G，调度器需要保证所有的 P 都有 G 执行，以保证并行度。如下图：

![调度示意图2](D:\typora\code\go\协程基本概念\协程调度示意图2.png)

从图中我们可以看见，当前有两个 P，各自绑定了一个 M，并分别执行了一个 goroutine，我们还可以看见每个 P上还挂了一个 G  的队列，这个队列是代表私有的任务队列，它们实际上都是 runnable 状态的 goroutine。当使用go 关键字声明时，一个  goroutine 便被加入到运行队列的尾部。一旦一个 goroutine 运行到一个调度点，上下文便从运行队列中取出一个  goroutine，设置好栈和指令指针，便开始运行新的  goroutine。