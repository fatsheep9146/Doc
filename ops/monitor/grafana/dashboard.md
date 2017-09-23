Dashboard笔记

Dashboard是Grafana中最为重要的一类资源，是用户使用Grafana最重要的一个接口。下面将记录有关Dashboard的一些高级使用技巧。

## Templating

模板化技术(Templating)是Dashboard非常重要的一种技术方法，主要涉及到的功能就是Grafana提供的Variable功能。

Variable，通过字面意义就可以知道它不是固定的。通过为某一个Variable指明其计算的方法，就可以通过这个方法动态获取变量的值，而这些动态获取的变量的值，可以在Metrics Query里面，在Panel Tilte里面动态引用。从而达到模板化的目的。引用格式包括两种。
```
$varname
[[varname]]
```
变量在每一个Dashboard通常是以一个下拉菜单的形式展示出来的。

Variable的基本信息包括几个

* Name: 变量名
* Lable: 变量在Dashboard下拉菜单的菜单名
* Hide: 是否隐藏下拉菜单
* Type: 变量类型

### 变量类型

变量包含很多种类型。

其中最重要的莫过于类型为Query类型的变量。这类变量是通过编写对指定data source的query语句来获取的变量。

query类型的变量也包括了很多选项：

* DataSource:
* Refresh:
* Query:
* Regex:
* Sort:

另外还有其他的一些重要选项

* Mult-value
* Include All option    
* Custom All options





