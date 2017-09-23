## JSON对引用类型的处理

在Go语言中包含几种引用类型(reference type)，这些类型的特点在于，如果仅仅采用var关键字声明一个引用类型的变量的话，那么这个变量的值为nil。

Go语言中包含的几种引用类型：

* pointer
* slice
* map

所以如果Json要解析的结构体中，包含引用类型的字段将会怎样进行处理呢？

处理逻辑如下：

首先根据JSON Raw Data中某一个字段的key的值，查找在目标结构体中是否有符合条件的字段。

如果找到，并且该字段的类型为引用类型的话，则首先通过new方法为这个字段分配内存。然后把Raw Data中该字段的值一一填入该字段中。

比如下述类型：
```
type FamilyMember struct {
    Name    string
    Parents []string
}
var m FamilyMember
err := json.Unmarshal(b, &m)
```

此时在Unmarshal时，首先找到Parents字段，根据它的类型为[]string，则为这个字段首先申请空间，然后在把解析出的值填进去。


### 高级使用场景

这是由于这个功能，使我们能够完成一个比较高级的场景，即已知多种可能JSON的对象类型时，比如如下类型：

```
type IncomingMessage struct {
    Cmd *Command
    Msg *Message
}
```

在这类型中包含两种子类型，而发送过来的IncomingMessage对象只能是这两种之一，此时如果发送过来的Raw Data中包含的是Cmd字段，此时系统会为这个字段分配空间，并且进行赋值。

而其他没有提到的字段的值仍旧为nil。

## 参考引用

1. https://blog.golang.org/json-and-go


