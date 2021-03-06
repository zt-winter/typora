# 模块 module

一个基本模块的创建

```go
# cd ~/code/go
# mkdir greetings && cd greetings
# go mod init test.com/greetings
go: creating new go.mod: module test.com/greetings
# vim greetings.go
//函数首字母大写，表示该函数可以被其他包调用
package greetings

import "fmt"

func Hello(name string) string {
        message := fmt.Sprintf("Hi, %v. welcome!", name)
        return message
}

//需要有main package，但go mod init "模块名" 模块名任意
# go build 
# ./"模块名"
```

一个模块调用另一个模块

```go
# cd ~/code/go
# mkdir hello && cd hello
# go mod init test.com/hello
go: creating new go.mod: module test.com/hello
# vim hello.go
package main

import(
        "fmt"
        "test.com/greetings"
)

func main() {
        message := greetings.Hello("Gladys")
        fmt.Println(message)
}
```

​		上面这种包引用情况是，你将greetings里所有函数打包，然后形成greetings包上传到test.com中，然后在写hello时要调用greetings中的函数，就直接从网上下载，然后使用。但很多时候，一个大项目中许多小项目要互相引用，但有要引用的项目还没有上线，或者两个项目都在一个主机上，那么该如何引用其中的函数呢？这里提供方法，就是相对于本项目，指出要引用的包的路径。

​		这里我们要引用greetings中函数

```go
# tree
.
├── greetings
│   ├── go.mod
│   └── greetings.go
├── hello
│   ├── go.mod
│   └── hello.go
└── pkg
    └── mod
        └── cache
            └── lock
/* 	在hello目录的命令提示符下执行下列命令
	将go.mod文件中test.com/greetings替换为../greetings
	当前目录~/code/go/hello，那么../greetings就是
	~/code/go/greetings目录
*/
# go mod edit -replace test.com/greetings=../greetings

/*	在hello目录的命令提示符下执行下列命令
	该命令会在刷新同步test.com/hello模块的依赖，
	自动添加不全所需要的代码
*/
# go mod tidy
go: found test.com/greetings in test.com/greetings v0.0.0-00010101000000-000000000000

//	这之后go.mod文件的内容
module test.com/hello

go 1.17

replace test.com/greetings => ../greetings

require test.com/greetings v0.0.0-00010101000000-000000000000
# go run .
Hi, Gladys. welcome!
```

这里我们继续添加一个模块，模块位于hello模块中。目录结构如下

```
# mkdir ./hello/submodule
# tree
├── greetings
│   ├── go.mod
│   └── greetings.go
├── hello
│   ├── go.mod
│   ├── hello
│   ├── hello.go
│   └── submodule
└── pkg
    └── mod
        └── cache
            └── lock
```

```go
//	我们在submodule文件夹下开始新的模块的建立
# go mod init test.com/submodule
# vim submodule.go
package submodule

import "fmt"

func Test() int {
        fmt.Println("This is a test module")
        return 1
}
//	然后我们回到hello目录下
# vim hello.go
package main

import(
        "fmt"
        "test.com/greetings"
        "test.com/submodule"
)

func main() {
        message := greetings.Hello("Gladys")
        fmt.Println(message)
        submodule.Test()
}
# go mod edit -replace test.com/submodule=./submodule
# go mod tidy
go: found test.com/submodule in test.com/submodule v0.0.0-00010101000000-000000000000
# go run .
Hi, Gladys. welcome!
This is a test module
```

## 在同一个项目下

**注意**：在一个项目（project）下我们是可以定义多个包（package）的。

### 目录结构

现在的情况是，我们在`moduledemo/main.go`中调用了`mypackage`这个包。

```bash
moduledemo
├── go.mod
├── main.go
└── mypackage
    └── mypackage.go
```

### 导入包

这个时候，我们需要在`moduledemo/go.mod`中按如下定义：

```go
module moduledemo

go 1.14
```

然后在`moduledemo/main.go`中按如下方式导入`mypackage`

```go
package main

import (
    "fmt"
    "moduledemo/mypackage"  // 导入同一项目下的mypackage包
)
func main() {
    mypackage.New()
    fmt.Println("main")
}
```

### 举个例子

举一反三，假设我们现在有文件目录结构如下：

```bash
└── bubble
    ├── dao
    │   └── mysql.go
    ├── go.mod
    └── main.go
```

其中`bubble/go.mod`内容如下：

```go
module github.com/q1mi/bubble

go 1.14
```

`bubble/dao/mysql.go`内容如下：

```go
package dao

import "fmt"

func New(){
    fmt.Println("mypackage.New")
}
```

`bubble/main.go`内容如下：

```go
package main

import (
    "fmt"
    "github.com/q1mi/bubble/dao"
)
func main() {
    dao.New()
    fmt.Println("main")
}
```

## 不在同一个项目下

### 目录结构

```bash
├── moduledemo
│   ├── go.mod
│   └── main.go
└── mypackage
    ├── go.mod
    └── mypackage.go
```

### 导入包

这个时候，`mypackage`也需要进行module初始化，即拥有一个属于自己的`go.mod`文件，内容如下：

```go
module mypackage

go 1.14
```

然后我们在`moduledemo/main.go`中按如下方式导入：

```go
import (
    "fmt"
    "mypackage"
)
func main() {
    mypackage.New()
    fmt.Println("main")
}
```

因为这两个包不在同一个项目路径下，你想要导入本地包，并且这些包也没有发布到远程的github或其他代码仓库地址。这个时候我们就需要在`go.mod`文件中使用`replace`指令。

在调用方也就是`packagedemo/go.mod`中按如下方式指定使用**相对路径**来寻找`mypackage`这个包。

```go
module moduledemo

go 1.14


require "mypackage" v0.0.0
replace "mypackage" => "../mypackage"
```

### 举个例子

最后我们再举个例子巩固下上面的内容。

我们现在有文件目录结构如下：

```bash
├── p1
│   ├── go.mod
│   └── main.go
└── p2
    ├── go.mod
    └── p2.go
```

`p1/main.go`中想要导入`p2.go`中定义的函数。

`p2/go.mod`内容如下：

```go
module liwenzhou.com/q1mi/p2

go 1.14
```

`p1/main.go`中按如下方式导入

```go
import (
    "fmt"
    "liwenzhou.com/q1mi/p2"
)
func main() {
    p2.New()
    fmt.Println("main")
}
```

因为我并没有把`liwenzhou.com/q1mi/p2`这个包上传到`liwenzhou.com`这个网站，我们只是想导入本地的包，这个时候就需要用到`replace`这个指令了。

`p1/go.mod`内容如下：

```go
module github.com/q1mi/p1

go 1.14


require "liwenzhou.com/q1mi/p2" v0.0.0
replace "liwenzhou.com/q1mi/p2" => "../p2"
```
