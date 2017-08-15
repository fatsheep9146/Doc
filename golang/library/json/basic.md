# Golang JSON处理机制

Go中使用"encoding/json"原生库处理JSON的编解码工作。

## 基础使用

其中最重要的两个方法就是编码函数"Marshal"和解码函数"Unmarshal"。

其中要注意的是这两个函数的函数签名:

```
func Marshal(v interface{}) ([]byte, error)

func Unmarshal(data []byte, v interface{}) error
```

所以通过最简单的使用例子即可知道Json库的最基本使用方法。见附录1

注意几点：

1. Marshal传入的是类型对象，Unmarshal传入的是类型对象指针。
2. 传入的编码后的信息是[]byte类型

在Unmarshal时，json库会根据传入的指针所指向的对象类型的结构来给这个对象赋值，遵循以下规则

1. 只查找结构体中**导出的域**
2. 首先查找导出域中带有和key的值完全一样tag的域
3. 如果2没有，查找导出域的名字和key的值完全一样的域
4. 如果3没有，查找导出域的名字经过大小写变换可以变成和key的值一样的域
5. 如果都没有则这个key会被忽略

注 \*\*\*\*\*\* 从5中可以看出json在Unmarshal时会忽略[]byte文本中多余的key字段。

## JSON结构未知场景

前面的例子讲解的都是对JSON对象的结构是已知的情况，如果对象结构是未知的又会怎么办呢？

此时我们可以采用interface{}类型作为Unmarshal的第二个参数传递进去，只要第一个参数中存放的是一个有效的JSON文本，那么就会被正确解析到第二个interface{}对象中去。

此时我们可以通过类型断言的方式来具体访问整个interface{}类型对象的内部结构。

注意，由于对于原始的JSON结构是未知的，所以通过这种方式解析出来的JSON结构体中，每个字段的类型将采用统一的规则来进行转换。规则如下：

JSON boolean -> bool
JSON number  -> float64
JSON string  -> string
JSON null    -> nil
JSON array   -> []interface{}
JSON map     -> map[string]interface{}

附录2中有例子函数


## 附录

### 附录1. JSON处理最基本实例

```
type Message struct {
    Name string `json:"name"`
    Body string `json:"name"`
}

func main() {
    m := Message{
        Name: "fatsheep9146",
        Body: "So fat",
    }

    b, err := json.Marshal(m) // b = `{"Name":"fatsheep9146", "Body":"So fat"}`
    if err != nil {
        fmt.Errorf("%v", err)
    }

    var n Message
    err = json.Unmarshal(b, &n)
    if err != nil {
        fmt.Errorf("%v", err)
    }
}
```

### 附录2 JSON处理未知JSON类型的情况

```
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

func main () {
    var f interface{}
    err := json.Unmarshal(b, &f)
    m := f.(map[string]interface{})  // type assertion
    for k, v := range m {
    switch vv := v.(type) {
        case string:
            fmt.Println(k, "is string", vv)
        case int:
            fmt.Println(k, "is int", vv)
        case []interface{}:
            fmt.Println(k, "is an array:")
            for i, u := range vv {
                fmt.Println(i, u)
            }
        default:
            fmt.Println(k, "is of a type I don't know how to handle")
    }
}
}
```

## 参考引用：

1. https://blog.golang.org/json-and-go



