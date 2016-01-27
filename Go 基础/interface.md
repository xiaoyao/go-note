# interface

## 类型断言
Go语言里面有一个语法，可以直接判断是否是该类型的变量： value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。

eg：
```go
for index, element := range list {

          if value, ok := element.(int); ok {

              fmt.Printf("list[%d] is an int and its value is %d\n", index, value)

          } else if value, ok := element.(string); ok {

              fmt.Printf("list[%d] is a string and its value is %s\n", index, value)

          } else if value, ok := element.(Person); ok {

              fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)

          } else {

              fmt.Printf("list[%d] is of a different type\n", index)

          }

}
```
另一个实现：用switch：
```go
for index, element := range list{

          switch value := element.(type) {

              case int:

                  fmt.Printf("list[%d] is an int and its value is %d\n", index, value)

              case string:

                  fmt.Printf("list[%d] is a string and its value is %s\n", index, value)

              case Person:

                  fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)

              default:

                  fmt.Println("list[%d] is of a different type", index)

          }

}
```