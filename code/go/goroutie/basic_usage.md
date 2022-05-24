# 协程的基本使用



### 简单的例子

```go
package main

import(
    "runtime"
)

func main() {
    runtime.GOMAXPROCS(16)
    //共计有14个task 协程在执行
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    go task()
    
    select{}
}

func task() {
    for {
    }
}
```

执行后，top命令如下，test命令占用了1401%的CPU，这里是因为top是只计算单核的，也可以说有14个逻辑cpu是占用满的

![top](D:\typora\code\go\goroutie\top.png)

下面用top查看单个cpu的情况

![top单核](D:\typora\code\go\goroutie\top_onecore.png)

可以看到里面有14个逻辑核心使用率几乎都达到了100%。

然后可以从系统管理器查看使用情况

![系统管理](D:\typora\code\go\goroutie\system_control.png)



### 协程的基本使用与控制

```
package main
import (
	"fmt"
)

func A(i int) {
	fmt.Println("A function")
}
func main() {
	fmt.Println("main function")
	go A(1)
    fmt.Println("main function is over")
}
```

```
$ ./test
main function
main function is over
```

在默认情况下，每个独立的 Go 应用运行时就创建了一个 Go 协程，其 `main` 函数就在这个 Go 协程中运行，这个 Go 协程就被称为 `go 主协程（main Goroutine)`。Go调度器在主协程执行完之前不会将控制权移交给A协程。同时一旦主协程执行完毕，整个程序就会终止，调度器就没有时间留给A协程去运行。

在main函数末尾加上`fmt.Println('main function is over')`前面加上`time.Sleep(10*time.Millisecond)`后再编译执行

```shell
$ ./test
main function
$ ./test
main function
A function
main function is over
```

然后在A函数中加入`time.Sleep(1*time.Millisecond)`编译执行，发现结果没有变化。但如果将休眠时间改为`time.Sleep(10*time.Millisecond)`。

```shell
$ ./test
main function
main function is over
```

一般子协程的执行时间是无法事先估计的，所以实际要达到上述效果，一般通过三种方式实现

1. sync.WaitGroup的三个方法Add(),Done(),Wait()
2. 带buffer的channel来控制
3. sync.Cond

