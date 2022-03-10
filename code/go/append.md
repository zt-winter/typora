# append函数

append函数原型

```go
func append(slice []Type, elems ...Type) []Type
```

内置函数append可以在切片末尾添加元素。如果切片有足够的空间，后面的空间会分配给新的元素。如果没有足够空间，就会一个新的隐含数组会被分配出来，然后将所有的元素复制到新的隐含数组中，然后返回这个隐含数组的头部指针（或者说新的切片）。

```go
package main

import(
    "fmt"
    "unsafe"
)

func main(){
	var slice []int = []int{1, 2, 3}
	var slices [][]int = [][]int{{3, 4}, {5, 6}, {7, 8}}
	fmt.Printf("go中一个字节8位\n")
	fmt.Printf("字节数为%d\n", unsafe.Sizeof(slice[0]))
    fmt.Printf("slices的空间为%d\n", cap(slices[0]))
	fmt.Printf("%p %p %p\n", slices[0], slices[1], slices[2])
	slices = append(slices, slice)
	fmt.Printf("%p %p %p %p\n", slices[0], slices[1], slices[2], slices[3])
	var slicesTwo [][]int = append([][]int{slice}, slices...)
	fmt.Printf("%p %p %p %p\n", slicesTwo[0], slicesTwo[1], slicesTwo[2], slicesTwo[3])
}
```

上面是append正确的使用方法。` slices = append(slices, slice)`是将slice切片添加到slices二维切片的后面。如果要将slice添加到slices的前面，应该是

` var sliceTwo [][]int = append([][]int{slice}, slices...)`

` [][]int{slice}`是新建了一个二维切片，然后切片的初始化了一个元素（一个一维切片）。

` slices...`是将slices中每一个元素全部列出来作为参数，也就是` elem ...Type`。

最后slices和sliceTwo的值为

```
slices->
[0]:[3,4]
[1]:[5,6]
[2]:[7,8]
[3]:[1,2,3]
sliceTwo->
[0]:[1,2,3]
[1]:[3,4]
[2]:[5,6]
[3]:[7,8]
[4]:[1,2,3]
```

```
//程序输出
go中一个字节8位
字节数为8
0xc0000160a0 0xc0000160b0 0xc0000160c0
0xc0000160a0 0xc0000160b0 0xc0000160c0 0xc00000e168
0xc00000e168 0xc0000160a0 0xc0000160b0 0xc0000160c0
```

0xc0000160a0 + 



