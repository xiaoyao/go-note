# Go 基础

[Go语言圣经（中文版）](http://golang-china.github.io/gopl-zh/index.html)


## make、new操作
    make用于内建类型（map、slice 和channel）的内存分配。

    new用于各种类型的内存分配。
    
    
内建函数new本质上说跟其它语言中的同名函数功能一样：new(T)分配了零值填充的T类型的内存空间，并且返回其地址，即一个*T类型的值。用Go的术语说，它返回了一个指针，指向新分配的类型T的零值。有一点非常重要：

> *new返回指针。*

内建函数make(T, args)与new(T)有着不同的功能，
    
    make只能创建slice、map和channel，并且返回一个有初始值(非零)的T类型，而不是*T。

本质来讲，导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。例如，一个slice，是一个包含指向数据（内部array）的指针、长度和容量的三项描述符；在这些项目被初始化之前，slice为nil。对于slice、map和channel来说，make初始化了内部的数据结构，填充适当的值。

>*make返回初始化后的（非零）值*

## string
string 类型不可修改

可以将 string 类型转换成 []byte 类型，进行修改操作。
```go
s := "hello"
c := []byte(s)  // 将字符串 s 转换为 []byte 类型
c[0] = 'c'
s2 := string(c)  // 再转换回 string 类型
fmt.Printf("%s\n", s2)

```

但 string 类型可以进行切片操作

```go
s := "hello"
s = "c" + s[1:] // 字符串虽不能更改，但可进行切片操作
```

多行字符串用“`”来声明：
```
m := `hello
    world`
```
“`” 括起的字符串为Raw字符串，即字符串在代码中的形式就是打印时的形式，它没有字符转义，换行也将原样输出。例如本例中会输出：
```
hello
    world
```

## iota枚举
Go里面有一个关键字iota，这个关键字用来声明enum的时候采用，它默认开始值是0，const中每增加一行加1：
```
const(
    x = iota  // x == 0
    y = iota  // y == 1
    z = iota  // z == 2
    w  // 常量声明省略值时，默认和之前一个值的字面相同。这里隐式地说w = iota，因此w == 3。其实上面y和z可同样不用"= iota"
)

const v = iota // 每遇到一个const关键字，iota就会重置，此时v == 0

const ( 
  e, f, g = iota, iota, iota //e=0,f=0,g=0 iota在同一行值相同
)

const （
    a = iota    a=0
    b = "B"
    c = iota    //c=2
    d,e,f = iota,iota,iota //d=3,e=3,f=3
    g //g = 4
）
```

## map
使用map过程中需要注意的几点： 
- map是无序的，每次打印出来的map都会不一样，它不能通过index获取，而必须通过key获取 - map的长度是不固定的，也就是和slice一样，也是一种引用类型 

- 内置的len函数同样适用于map，返回map拥有的key的数量 - map的值可以很方便的修改，通过numbers["one"]=11可以很容易的把key为one的字典值改为11 
- map和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制
- 
map的初始化可以通过key:val的方式初始化值，同时map内置有判断是否存在key的方式

通过delete删除map的元素：

```go
// 初始化一个字典
rating := map[string]float32{"C":5, "Go":4.5, "Python":4.5, "C++":2 }
// map有两个返回值，第二个返回值，如果不存在key，那么ok为false，如果存在ok为true
csharpRating, ok := rating["C#"]
if ok {
    fmt.Println("C# is in the map and its rating is ", csharpRating)
} else {
    fmt.Println("We have no rating associated with C# in the map")
}
delete(rating, "C")  // 删除key为C的元素
```
上面说过了，map也是一种引用类型，如果两个map同时指向一个底层，那么一个改变，另一个也相应的改变：

```go
m := make(map[string]string)    
m["Hello"] = "Bonjour"
m1 := m
m1["Hello"] = "Salut"  // 现在m["hello"]的值已经是Salut了
```