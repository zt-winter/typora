go语言中省略号一般用在三个地方

* 数组初始化中，用于编译器自动推断元素个数

```go
array := [...]int{1,2,3}
```

* 切片（数组指针）后面，用于表示将切片中每一个元素列出

```go
sliceThree := append(sliceTow, sliceOne...)
```

* 函数传递参数，用于表示参数的数量不固定，变长参数

```go
func functionOne(parms ...int){
	for i,v := range parms {
		fmt.Printf("%v %v\n", i, v)
	}
}
func main(){
	function(0,1,2)
}
```

```go
package main
import(
    "bytes"
    "fmt"
    "os/exec"
    "log"
)

func main() {
    args := []string{"/home/zt", "-l"}
    cmd := exec.Command("ls", args...)
    //cmd := exec.Command("ls", "/home/zt", "-l")
    var out bytes.Buffer
    cmd.Stdout = &out
    err := cmd.Run()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("\n%q\n", out.String())
}
```



