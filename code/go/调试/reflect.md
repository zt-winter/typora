获取变量类型的三种方法

package main
import (
    "fmt"
    "reflect"
)
func main() {
    var num float64 = 3.14

    // 方法1：
    println(reflect.TypeOf(num).Name())
     
    // 方法2：
    fmt.Println(reflect.TypeOf(num))
     
    // 方法3：
    fmt.Printf(`%T`, num)
}