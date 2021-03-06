# GO基本数据结构

* string

  * go语言的字符串无法直接修改每一个字符元素，只能通过重新构造新的字符串。

    ```
    package main
    import(
    	"fmt"
    )
    
    func main() {
    	test := "hello world"
    	test1 := []byte(test)
    	for i := 6; i < 8; i++ {
    		test1[i] = ' '
    	}
    	fmt.Println(string(test1))
    	fmt.Println(test)
    	test2 := []byte(test)
    	test2[0] = 'f'
    	test2[1] = 'u'
    	test2[2] = 'c'
    	test2[3] = 'k'
    	test2[4] = ' '
    	fmt.Println(string(test2))
    	
    hello   rld
    hello world
    fuck  world
    ```

* array && slice

  * 数组的数据类型是[length]int，length是一个确定的数字。数组是具有相同 唯一类型 的一组以编号且长度固定的数据项序列（这是一种同构的数据结构）；这种类型可以是任意的原始类型例如整型、字符串或者自定义类型。数组长度必须是一个常量表达式，并且必须是一个非负整数。数组长度也是数组类型的一部分，所以 [5] int s和 [10] int 是属于不同类型的。

  * 数组的长度一旦确定就不能更改，这与C/C++有很大的不同。

  ```
  //因此下面这种代码是错的，在编译环节就过不去
  func test(length int){
  	var array[length]int
  =================================
  func test(){
  	length := 10
  	var array[length]int
  ```
  
  * 切片不一样，切片本质是指针，切片的数据类型是[]int，切片的初始化一般通过数组创建` sliceOne := array[0:1]`，也可以直接初始化赋值` var sliceTwo []int = []int{10,11,12,13,14,15}`，还可以通过make创建，这个后面再说。
  
  下面的代码可以说明一些性质
  
  ```go
  package main
  import(
  	"fmt"
  	"reflect"
  )
  func main() {
  	var array = [5]int{0, 1, 2, 3, 4}
  	fmt.Printf("array: %p\n", &array)
  	var sliceOne = array[0:5]
  	fmt.Println(reflect.TypeOf(sliceOne))
  	fmt.Printf("sliceOne: %p\n", sliceOne)
  	for i := 0; i < len(sliceOne); i++ {
  		fmt.Printf("%d\n", sliceOne[i])
  	}
      sliceOne[4] = 100
      fmt.Printf("slicOne: %p\n", sliceOne)
  	sliceOne = append(sliceOne, 5)
  	fmt.Printf("sliceOne: %p\n", sliceOne)
  	for i := 0; i < len(sliceOne); i++ {
  		fmt.Printf("%d\n", sliceOne[i])
  	}
  	var sliceTwo []int = []int{10, 11, 12, 13, 14, 15}
  	fmt.Printf("sliceTwo: %p\n", sliceTwo)
  	sliceTwo = sliceOne
      fmt.Printf("sliceTwo: %p\n", sliceTwo)
  	for i := 0; i < len(sliceTwo); i++ {
  		fmt.Printf("%d\n", sliceTwo[i])
  	}
  }
  =========编译执行==========
  array: 0xc00000a4b0
  []int
  sliceOne: 0xc00000a4b0
  0
  1
  2
  3
  4
  sliceOne: 0xc00000a4b0, array[4]=100
  sliceOne: 0xc000010410
  0
  1
  2
  3
  100
  5
  sliceTwo: 0xc00000a510
  sliceTwo: 0xc000010410
  0
  1
  2
  3
  100
  5
  ```
  
  * 使用数组给切片sliceOne初始化时，sliceOne的值与&array的值是一样的，对sliceOne[i]中的值就行修改时，array中对应的值也会修改。但sliceOne的长度发生变化时，sliceOne的地址就会发生变化。相当于在新开的内存区域创建了一个新的C/C++数组。切片也可以互相赋值，如sliceTwo=sliceOne，但要注意，这样0xc00000a510的值就再也没有指针指向它，无法释放，造成了内存使用不安全。
	```
	// one是一个一维数组，ans是一个二维数组
	onecopy := make([]int, len(one))
	copy(onecopy, one)
	ans = append(ans, onecopy)
	//或者
	ans = append(ans, append([]int{}, path...))
	```
  
  * 二维切片需要注意的地方
  
  ```go
  var test = [][]int{{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}, {13, 14, 15, 16}}
  var testOne [][]int = test[1:4][:]
  /* testOne的值
  5, 6, 7, 8
  9, 10, 11, 12
  13, 14, 15, 16
  */
  
  var testTwo [][]int = testOne[1:3]
  /* testTwo的值
  9, 10, 11, 12
  13, 14, 15, 16
  */
  
  //想从4*4的数组中取中心的2*2的数组
  var testerror [][]int = test[1:4][1:4]
  //报错"runtime error: slice bounds out of range [:4] with capacity 3"
  
  /* 
  对二维切片不能同时取两次切片，取一次切片也是对test[]取切片。
  var testThree [][]int = test[1:4][1:3]也不是对test[][]同时取了两次切片
  实际上的操作是：
  var tmp [][]int = test[1:4]
  var testThree [][]int = tmp[1:3]
  这样可以理解testerror为什么会报错
  */
  //实现从4*4的数组中取中心的2*2的数组
  var testerror [][]int
  for i := 1; i < 3; i++ {
      testerror = append(testerror, test[i][1:3])
  ```


* 值得注意的符号
  * 分号;    GO与C语言一样，采用分号作为一条语句的结束，但GO的语法解析器自动在每一行的末尾添加分号，{除外。所以在就会有下面得到语法` if firstString, isOk = mapString[tmp]; isOk{`
  * 大括号{}    用于初始化
* hash表，字典，map

map的初始化

```go
mapInstance := make(map[typeOne]typeTwo)
mapInstance[typeOne] = typeTwo
//在这里typeOne就是key，typeTwo就是value
```

使用数组或者切片作为key   ` mp := map[[26]int][]string{}`。

```go
[26]int：数组作为key
[]string：字符串数组作为value
{}：map初始化
```

