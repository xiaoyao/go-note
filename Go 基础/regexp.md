# regexp

### regexp.Compile()
该函数将正则表达式编译成有效的可匹配格式。当输入的正则表达式不合法时，该函数会返回一个错误。

### regexp.MustCompile()
在程序源码中，大多数正则表达式是字符串字面值（string literals），因此regexp包提供了包装函数regexp.MustCompile检查输入的合法性。

```go
package regexp
func Compile(expr string) (*Regexp, error) { /* ... */ }
func MustCompile(expr string) *Regexp {
    re, err := Compile(expr)
    if err != nil {
        panic(err)
    }
    return re
}
```

> 包装函数使得调用者可以便捷的用一个编译后的正则表达式为包级别的变量赋值：

```go

var httpSchemeRE = regexp.MustCompile(`^https?:`) //"http:" or "https:"
```
显然，MustCompile不能接收不合法的输入
