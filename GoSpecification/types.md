# Types

常见的ASCII控制代码的转义方式：
```
\a      响铃
\b      退格
\f      换页
\n      换行
\r      回车
\t      制表符
\v      垂直制表符
\'      单引号 (只用在 '\'' 形式的rune符号面值中)
\"      双引号 (只用在 "..." 形式的字符串面值中)
\\      反斜杠
```
### 0.channel

channel跟string或slice有些不同，它在栈上只是一个指针，

实际的数据都是由指针所指向的堆上面。

跟channel相关的操作有：初始化/读/写/关闭。channel未初始化值就是nil，未初始化的channel是不能使用的。下面是一些操作规则：

- 读或者写一个nil的channel的操作会永远阻塞。
- 读一个关闭的channel会立刻返回一个channel元素类型的零值。
- 写一个关闭的channel会导致panic。



## 1.string
### string 类型不可修改

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

### 多行字符串用“`”来声明：
```
m := `hello
    world`
```
“`” 括起的字符串为Raw字符串，即字符串在代码中的形式就是打印时的形式，它没有字符转义，换行也将原样输出。例如本例中会输出：
```
hello
    world
```

内置的len函数可以返回一个字符串中的字节数目（不是rune字符数目），索引操作s[i]返回第i个字节的字节值
```go
s := "hello, world"
fmt.Println(len(s))     // "12"
fmt.Println(s[0], s[7]) // "104 119" ('h' and 'w')
```
第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节。

字符串的值是不可变的：一个字符串包含的字节序列永远不会被改变.

不变性意味如果两个字符串共享相同的底层数据的话也是安全的，这使得复制任何长度的字符串代价是低廉的.

** 字符串切片操作代价也是低廉的。在这两种情况下都没有必要分配新的内存**

***

![](string.png)

字符串在Go语言内存模型中用一个2字长的数据结构表示。它包含一个指向字符串存储数据的指针和一个长度数据。因为string类型是不可变的，对于多字符串共享同一个存储数据是安全的。

切分操作str[i:j]会得到一个新的2字长结构，一个可能不同的但仍指向同一个字节序列(即上文说的存储数据)的指针和长度数据。这意味着字符串切分可以在不涉及内存分配或复制操作。这使得字符串切分的效率等同于传递下标。
***

## 2.slice
Slice（切片）代表**变长的序列**，序列中每个元素都有相同的类型。

一个slice类型一般写作[]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已。

一个slice是一个数组某个部分的引用。

数组和slice之间有着紧密的联系。一个slice是一个轻量级的数据结构，提供了访问数组子序列（或者全部）元素的功能，而且slice的底层确实引用一个数组对象。一个slice由三个部分构成：指针、长度和容量。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数分别返回slice的长度和容量。

在内存中，它是一个包含3个域的结构体：指向slice中第一个元素的指针，slice的长度，以及slice的容量。

多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠。

slice的切片操作s[i:j]，其中0 ≤ i≤ j≤ cap(s)，用于创建一个新的slice，引用s的从第i个元素开始到第j-1个元素的子序列。新的slice将只有j-i个元素。如果i位置的索引被省略的话将使用0代替，如果j位置的索引被省略的话将使用len(s)代替。因此，months[1:13]切片操作将引用全部有效的月份，和months[1:]操作等价；months[:]切片操作则是引用整个数组。

如果切片操作超出```cap(s)```的上限将导致一个panic异常，但是超出len(s)则是意味着扩展了 slice，因为新 slice 的长度会变大。

和数组不同的是，slice之间不能比较，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较。
```go
func equal(x, y []string) bool {
    if len(x) != len(y) {
        return false
    }
    for i := range x {
        if x[i] != y[i] {
            return false
        }
    }
    return true
}
```
**slice唯一合法的比较操作是和nil比较.**


- **长度是下标操作的上界**，如x[i]中i必须小于长度。
- **容量是分割操作的上界**，如x[i:j]中j不能大于容量。

![](slice.png)

通常我们并不知道append调用是否导致了内存的重新分配，因此我们也不能确认新的slice和原始的slice是否引用的是相同的底层数组空间。同样，我们不能确认在原先的slice上的操作是否会影响到新的slice。因此，通常是将append返回的结果直接赋值给输入的slice变量：
```
runes = append(runes, r)
```
更新slice变量不仅对调用append函数是必要的，实际上对应任何可能导致长度、容量或底层数组变化的操作都是必要的。要正确地使用slice，需要记住尽管底层数组的元素是间接访问的，但是slice对应结构体本身的指针、长度和容量部分是直接访问的。要更新这些信息需要像上面例子那样一个显式的赋值操作。从这个角度看，slice并不是一个纯粹的引用类型，它实际上是一个类似下面结构体的聚合类型：
```
type IntSlice struct {
    ptr      *int
    len, cap int
}
```

## 3.iota枚举
Go里面有一个关键字iota，这个关键字用来声明enum的时候采用，它默认开始值是0，const中每增加一行加1：

```go
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

## 4.map
- map在底层是用哈希表实现的，

可以在 $GOROOT/src/pkg/runtime/hashmap.goc 找到它的实现。


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

## 5.nil

按照Go语言规范，任何类型在未初始化时都对应一个零值：布尔类型是false，整型是0，字符串是""，而指针，函数，interface，slice，channel和map的零值都是nil。

### interface{}
一个interface在没有进行初始化时，对应的值是nil。也就是说var v interface{}，

此时v就是一个nil。在底层存储上，它是一个空指针。与之不同的情况是，interface值为空。比如：
```go
var v *T
var i interface{}
i = v
```
此时i是一个interface，它的值是nil，但它自身不为nil。

### string & slice
string的空值是""，它是不能跟nil比较的。即使是空的string，它的大小也是两个机器字长的。slice也类似，它的空值并不是一个空指针，而是结构体中的指针域为空，空的slice的大小也是三个机器字长的。


### map
map也是指针，实际数据在堆中，未初始化的值是nil。

### channel
channel跟string或slice有些不同，它在栈上只是一个指针，实际的数据都是由指针所指向的堆上面。
