**0. 简单的哲学**

Go 的设计哲学之一就是追求简单，因此在命名上一样秉承着简单的总体原则**。**要想做好 Go 标识符命名（包括 package 命名)，至少要遵循两个原则：

- **简单且一致**
- **利用上下文辅助命名**

1. **区分大小写的语言**

命名规则涉及变量、常量、全局函数、结构、接口、方法等的命名。 Go语言从语法层面进行了以下限定：**任何需要对外暴露的名字必须以大写字母开头，不需要对外暴露的则应该以小写字母开头**。

当命名（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：SetupRouter，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出。

命名如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的。

**2. 包名称**

> The rule of thumb I follow is not, "what types should I put in this  package?". Instead the question I ask "what does service does package  provide?"

- 最好保持 package 的名字和目录保持一致。
- 尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。
- 包名应该为小写单词，尽量不要使用下划线或者混合大小写。
- 包名以及包所在的目录名，不要使用复数，例如，是net/utl，而不是net/urls。
- 不要用 common、util、shared 或者 lib 这类宽泛的、无意义的包名。
- 包名要简单明了，例如 net、time、log。

```go
package controller
```

**3. 文件命名**

尽量采取有意义的文件名，简短，有意义，应该为小写单词，使用下划线分隔各个单词。

```go
routers.go
user_controller.go
```

**4. 结构体命名**

采用驼峰命名法，首字母根据访问控制大写或者小写。

```go
type UserInfo struct {
	Name      string `gorm:"type:varchar(20);not null"`
	Telephone string `gorm:"type:varchar(11);not null;unique"`
	Password  string `gorm:"size:255;not null"`
}
```

**5. 接口命名**

在 Go 语言中 interface  名字仍然以单个词为优先。命名规则基本和上面的结构体类型类似。对于拥有唯一方法或通过多个拥有唯一方法的接口组合而成的接口，Go  语言的惯例是一般用"方法名+er"的方式为 interface 命名，例如 Reader , Writer 。

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

**6. 变量命名**

和结构体类似，变量名称一般遵循驼峰法，首字母根据访问控制原则大写或者小写，但遇到特有名词时，需要遵循以下规则：

如果变量为私有，且特有名词为首个单词，则使用小写，如 appService；

若变量类型为 bool 类型，则名称应以 Has, Is, Can 或 Allow 开头。

```go
var isExist bool
var hasConflict bool
var canManage bool
```

**7. 方法的接收器命名**

下面的代码里那个附加的参数p，叫做方法的接收器（receiver）

在Go语言中，我们并不会像其它语言那样用this或者self作为接收器；我们可以任意的选择接收器的名字。由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。

```go
type Point struct{ X, Y float64 }

func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

**8. 变量名字中不要带有类型信息**

```go
 userSlice []*User         // bad
 users     []*User         // good
```

不过可能会有这样的质疑：userSlice 中的类型信息可以告诉我们变量所代表的底层存储是一个切片，这样便可以在 userSlice 上应用切片的各种操作了。提出这样质疑的开发者显然忘记了一条编程语言命名的通用惯例：**保持变量声明与使用之间的距离越近越好或者说将变量声明在第一次使用该变量之前**。这个惯例与 Go 核心团队的安德鲁·杰拉德曾提到的一种说法：**“一个名字的声明和使用之间的距离越大，这个名字的长度就越长”**异曲同工。如果在一屏之内能看到 users 的声明，那 Slice 这个类型信息显然是不必放在变量的名称中了。

**9. 保持简短命名变量含义上的一致性**

Go 语言中有大量的单字母或单个词或缩写命名的简短命名变量，有人可能会质疑简短命名变量会带来可读性上的下降。Go 语言建议通过保持一致性来维持可读性。一致意味着代码中相同或相似的命名所传达的含义是相同或相似的，这样便于代码阅读者或维护者猜测出变量的用途。

```go
变量v, k, i的常用含义：

	// 循环语句中的变量
	for i, v := range s { ... } // i: 下标变量； v：元素值
	for k, v := range m { ... } // k: key变量；v: 元素值
	for v := range r { // channel ... } // v: 元素值

	// if、switch/case分支语句中的变量
	if v := mimeTypes[ext]; v != "" { } // v: 元素值
	switch v := ptr.Elem(); v.Kind() {  
		... ...
	}

	case v := <-c: // v: 元素值

	// 反射的结果值
	v := reflect.ValueOf(x)

变量t的常用含义：

	t := time.Now() // 时间
	t := &Timer{}// 定时器
	if t := md.typemap[off]; t != nil { }// 类型

变量b的常用含义：

	b := make([]byte, n) // byte切片
	b := new(bytes.Buffer) // byte缓存
```

Go 中的变量名应该短而不是长。对于作用域有限的局部变量尤其如此。更喜欢 c 而不是 lineCount。更喜欢 i 而不是 sliceIndex。

在相对简单（对象数量少、针对性强）的环境中，可以将一些名称由完整单词简写为单个字母，例如：user 可以简写为 u；userID 可以简写 uid。

**10. Error 的命名**

Error 类型应该写成 FooError 的形式。

```go
type ExitError struct {  
 // ....
}
```

Error 变量写成 ErrFoo 的形式。

```go
var ErrFormat = errors.New("unknown format")
```