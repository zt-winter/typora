# 内置函数

[参考文章](https://zhuanlan.zhihu.com/p/76524813)

universe.go源文件定义了go内置函数列表，Main函数调用initUniverse，进而调用lexinit对builtinpkg进行了初始化。SetSubOp调用会将Node结构(下文给出)中的Op设置为对应的值，用于标识该Node结构为内置函数。

```var builtinFuncs = [...]struct {
var builtinFuncs = [...]struct {
    name string
    op   Op
}{
    {"append", OAPPEND},
    {"cap", OCAP},
    {"close", OCLOSE},
    {"complex", OCOMPLEX},
    {"copy", OCOPY},
    {"delete", ODELETE},
    {"imag", OIMAG},
    {"len", OLEN},
    {"make", OMAKE},
    {"new", ONEW},
    {"panic", OPANIC},
    {"print", OPRINT},
    {"println", OPRINTN},
    {"real", OREAL},
    {"recover", ORECOVER},
}

func lexinit(){
    for _, s := range builtinFuncs {
        s2 := builtinpkg.Lookup(s.name)
        s2.Def = asTypesNode(newname(s2))
        asNode(s2.Def).SetSubOp(s.op)
    }
}
```

- `make` 的作用是初始化内置的数据结构，也就是我们在前面提到的切片、哈希表和 Channel[2](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-make-and-new/#fn:2)；
- `new` 的作用是根据传入的类型分配一片内存空间并返回指向这片内存空间的指针[3](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-make-and-new/#fn:3)；

```go
slice := make([]int, 100)
slice := make([]int, len)
slice := make([]int, len, cap)
hash := make(map[int]bool, 10)
ch := make(chan int, 5)
```

1. `slice` 是一个包含 `data`、`cap` 和 `len` 的结构体 [`reflect.SliceHeader`](https://draveness.me/golang/tree/reflect.SliceHeader)；
2. `hash` 是一个指向 [`runtime.hmap`](https://draveness.me/golang/tree/runtime.hmap) 结构体的指针；
3. `ch` 是一个指向 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 结构体的指针；

```go
i := new(int)
=============
var v int
i := &v
```