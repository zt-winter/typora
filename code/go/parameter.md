# 参数传递

## GO中参数传递只有值传递，没有参数传递



### slice map chan  在使用时都是指针类型

主函数调用test函数，在函数传递参数时，` func test( sliceA  []int) `，sliceA就是指针，它和原切片的指针不是同一个东西，两个指针指向同一个slice，但所在的位置不同。由于slice创建时，自身就有cap和len两个参数。如果只是对原有slice中数值进行修改，那么是没有问题的，主函数中slice也会修改。但如果进行扩容操作，如append，如果cap改变了，那么就会在新的slice上进行操作，返回新的slice地址。主函数中的slice地址就不会修改。

如果传递的参数是slice的地址，` func test ( sliceA *[]int)`。在进行扩容操作后，新的slice地址也会赋值给slice，相当于主函数中slice指针指向了新的slice地址。

所以 __在GO中没有按参数传递，只有指针的副本传递和值的副本传递__。

