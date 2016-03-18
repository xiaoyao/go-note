# defer

defer用于资源的释放，会在函数返回之前进行调用。

**尤其是跟带命名的返回参数一起使用时,很容易出错。**
## defer 大坑

例1：
```go

func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}
```
>result:1

例2：
```go

func f() (r int) {
     t := 5
     defer func() {
       t = t + 5
     }()
     return t
}
```
>result: 5

例3：

```go
func f() (r int) {
    defer func(r int) {
          r = r + 5
    }(r)
    return 1
}
```
>result: 1

它改写后变成：
```
func f() (r int) {
     r = 1  //给返回值赋值
     func(r int) {        //这里改的r是传值传进去的r，不会改变要返回的那个r值
          r = r + 5
     }(r)
     return        //空的return
}
```
**改的r是传值传进去的r，不会改变要返回的那个r值.**所以这个例子的结果是1。


defer是在return之前执行的。这个在 官方文档中是明确说明了的。

要使用defer时不踩坑，最重要的一点就是要明白，**return xxx这一条语句并不是一条原子指令!**


## 函数返回的过程：
- 先给返回值赋值，
- 然后调用defer表达式，
- 最后才是返回到调用函数中。

**defer表达式可能会在设置函数返回值之后，在返回到调用函数之前，修改返回值，使最终的函数返回值与你想象的不一致。**

defer确实是在return之前调用的。但表现形式上却可能不像。本质原因是return xxx语句并不是一条原子指令，defer被插入到了赋值 与 ret之间，因此可能有机会改变最终的返回值。


## defer的实现

defer关键字的实现跟go关键字很类似，不同的是它调用的是runtime.deferproc而不是runtime.newproc。

在defer出现的地方，插入了指令call runtime.deferproc，然后在函数返回之前的地方，插入指令call runtime.deferreturn。

普通的函数返回时，汇编代码类似：

add xx SP
return
如果其中包含了defer语句，则汇编代码是：

call runtime.deferreturn，
add xx SP
return
goroutine的控制结构中，有一张表记录defer，调用runtime.deferproc时会将需要defer的表达式记录在表中，而在调用runtime.deferreturn的时候，则会依次从defer表中出栈并执行。



***
## 例子：

```go
func whoFirst1() int { //0
	result := 0
	defer func() {
		result++
	}()
	return result
}

//-------------------------------------

func whoFirst2() (r int) { //5
	result := 5
	defer func() {
		result += 5
	}()
	return result
}

func whoFirst21() (result int) { //10
	result = 5
	defer func() {
		result += 5
	}()
	return result
}

func whoFirst22() int { //5
	result := 5
	defer func() {
		result += 5
	}()
	return result
}

//-------------------------------------

func whoFirst3() (result int) { //1
	defer func(result int) {
		result = result + 5
	}(result)
	return 1
}

func whoFirst31() (result int) { //0
	defer func(result int) {
		result = result + 5
	}(result)
	return result
}

func whoFirst32() (result int) { //5
	defer func() {
		result = result + 5
	}()
	return
}

func whoFirst33() (result int) { //6
	defer func() {
		result = result + 5
	}()
	return 1
}
```