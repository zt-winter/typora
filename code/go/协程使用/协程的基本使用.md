# 协程的基本使用

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

![top](D:\typora\code\go\协程使用\top命令.png)

下面用top查看单个cpu的情况

![top单核](D:\typora\code\go\协程使用\top单核.png)

可以看到里面有14个逻辑核心使用率几乎都达到了100%。

然后可以从系统管理器查看使用情况

![系统管理](D:\typora\code\go\协程使用\系统管理.png)

