## std::cmp

### 排序和比较功能

本模块包含用于排序和比较值的各种工具。

- `Eq`和`PartialEq` trait 允许定义两个值之间全部和部分相等，实现它们会重载`==`和`！=`运算符。
- `Ord`和`PartialOrd`定义值之间的总排序和部分排序，实现它们会重载<、<=、>和>=运算符。
- `Ordering`是由Ord和PartialOrd的主要函数返回的枚举，它描述了一个顺序。
- `Reverse` struct  反转顺序
- `max`,`min`是建立在Ord基础上的函数

### Macro

Eq 派生宏，生成一个trait Eq的impl。

Ord  派生宏，生成特征Ord的impl。

PartialEq   派生宏，生成trait PartialEq的impl。

PartialOrd   派生宏，生成trait PartialOrd的impl.