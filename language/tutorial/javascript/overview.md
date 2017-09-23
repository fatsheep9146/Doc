Javascript

基础：

js四种原生数据类型

- strings: 单引号 双引号均可
- number: 数组类型，可以带小数点
- boolean: 布尔类型
- null: null类型

四种数据运算

+ - * /

每一种原生类型都带有一定的原生性质，可以通过'.'操作符访问

"hello".length

每一种原生类型都带有一些原生方法，也可以通过'.'操作符访问

'Hello'.toUpperCase()

library:
不需要创建实例对象就可以调用的方法。

注释:
//  /* */

变量声明：

```
let var1 = val1 
const con2 = val2  //创建的是常量变量，内容不能被改变
```
注：如果let定义变量但是没有赋值，则会被自动分配undefined值，这是第五种原生类型值

字符串拼接

1.
let str = ""
let str1 = ""
str = str1 + "string2"

2. ES6 
let myName = "zzq";
let myCity = "Shanghai";
console.log(`My name is ${myName}, My favorite city is ${myCity}`);


