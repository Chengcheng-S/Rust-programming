# std::ops

可重载运算符

实现该模块的trait，可重载运算符

> 加法运算符（+）可以通过Add trait重载，但由于赋值运算符（=）没有支持特征，因此无法重载其语义

此模块不提供任何创建新的运算符的机制，如果需要无尾重载或自定义运算符，应该考虑使用宏或编译器插件来扩展Rust的语法。

许多运算符按值取操作数。在涉及内置类型的非泛型上下文中，这通常不是问题。但是，在泛型代码中使用这些运算符需要注意是否必须重用值，而不是让运算符使用它们。一种选择是偶尔使用克隆。另一个选择是依赖于为引用提供附加运算符实现的类型。

```rust
use std::ops::{Add, Sub};

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {x: self.x + other.x, y: self.y + other.y}
    }
}

impl Sub for Point {
    type Output = Point;

    fn sub(self, other: Point) -> Point {
        Point {x: self.x - other.x, y: self.y - other.y}
    }
}
```

> Fn、FnMut和FnOnce特征由可以像函数一样调用的类型来实现。注意Fn取&self，FnMut取&mut self，FnOnce取self。它们对应于可在实例上调用的三种方法：按引用调用、按可变引用调用和按值调用。这些特征最常见的用法是充当以函数或闭包作为参数的高级函数的边界。

### traits

| Add          | 加法运算符 `+`                                            |
| ------------ | --------------------------------------------------------- |
| AddAssign    | 加法赋值运算符`+=`                                        |
| BitAnd       | 位与运算符`&`                                             |
| BitAndAssign | 位与赋值运算符`&=`                                        |
| BitOr        | 按位或运算符 `|`                                          |
| BitOrAssign  | 按位或赋值运算符  `|=`                                    |
| BitXor       | 按位异或运算符`^`                                         |
| BitXorAssign | 按位异或赋值运算符`^=`                                    |
| DerefMut     | 可变解引用运算符`*a=3`                                    |
| Deref        | 解引用运算符   `*a`                                       |
| Div          | 除法运算符`/`                                             |
| DivAssign    | 除法赋值运算符`/=`                                        |
| Drop         | 析构函数中的自定义代码                                    |
| Fn           | 使用不可变接收器的调用运算符的版本                        |
| FnMut        | 使用可变接收器的调用运算符的版本。                        |
| FnOnce       | 接受按值接收器的调用运算符版本。                          |
| Index        | 用于不可变的上下文索引操作`container[index]`              |
| IndexMut     | 可变上下文中的索引操作`Container[index]`                  |
| Mul          | 乘法运算符`*`                                             |
| MulAssign    | 乘法赋值运算符`*=`                                        |
| Neg          | 一元否定运算符`-`                                         |
| Not          | 一元逻辑否定运算符`!`                                     |
| RangeBounds  | Rust的内置范围类型，由range语法生成 `..a`,`..=b`  `e..=a` |
| Rem          | 取余运算符`%`                                             |
| RemAssign`   | 取余赋值运算符 `%=`                                       |

| Shl             | 左移操作符`<<`，注：由于此特征是针对具有多个右侧类型的所有整数类型实现的，Rust的类型检查器对<<<有特殊处理，将整数运算的结果类型设置为左侧操作数的类型。这意味着，虽然a<<b和a.shl（b）从评估的角度来看是一个相同的，但是在类型推理方面它们是不同的。 |
| --------------- | ------------------------------------------------------------ |
| ShlAssign       | 左移指定运算符`<<=`                                          |
| Shr             | 右移操作符，注：由于此特征是针对具有多个右侧类型的所有整数类型实现的，因此Rust的类型检查器对>>>有特殊处理，将整数运算的结果类型设置为左侧操作数的类型。这意味着，虽然a>>b和a.shr（b）从求值的角度来看是一个相同的，但在类型推断方面它们是不同的。 |
| ShrAssign       | 右移指定运算符 >>=                                           |
| Sub             | 减法运算符`-`                                                |
| SubAssign       | 减法赋值运算符`-=`                                           |
| CoerceUnsized   | 试验阶段： trait  指示这是一个指针或一个的包装器，其中可以对指针对象执行取消大小调整。 |
| DispatchFromDyn | 试验阶段：对象安全性，用于检查方法的接收方法类型是否可以被调度 |
| Generator       | 实验： 由内置生成器类型实现的trait                           |
| Try             | 试验阶段： 定制`?`的行为                                     |

