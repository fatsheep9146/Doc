Go JSON使用教程

JSON是一种在现阶段被广泛使用的数据交换格式，自然而然，go语言也提供了encoding/json库，用来支持对JSON这种数据格式的处理。

针对JSON处理库encoding/json来说，其中有两个最重要的方法，会贯穿整个教程的始末。也是进行JSON数据处理最重要的两个函数。

我们可以通过手册看到两个函数的签名：

```
func Marshal(v interface{}) ([]byte, error)

func Unmarshal(data []byte, v interface{}) error

```

很显然Marshal函数是将程序中的一个对象(一般是结构体对象)转换为一个符合JSON语法的字符串。
而Unmarshal很显然是将一个符合JSON语法的字符串，转换为一个程序中的结构体对象。

即Marshal做的事情是，将Person类型的结构体对象Xiaoming
```
type Person struct {
    Name string  `json:"name"`
    Age  uint    `json:"age"`
}

Xiaoming := Person{
    Name: "Xiaoming",
    Age:  26,
}

====>  '{ "name":"Xiaoming", "age":26  }'
```

当然结构体的结构也不能随意的定义，真实的使用场景也不是这样理想，所以这个时候我们必须理解一下Marshal, Unmarshal在真正运行过程中真正遵循的转换原则。可以用以下几条来概括。

