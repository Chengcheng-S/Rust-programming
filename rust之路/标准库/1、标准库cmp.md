# cmp



## std::cmp

本模块包含用于**排序和比较值**的各种工具

- Eq和PrritalEq允许分别定义值之间的全部和部分相等的特性，实现他们会**重载and运算符以及== !=**

- Ord和PartialOrd，允许分别定义值之间的全部和部分顺序的特征，**实现他们会重载“，，，”   以及< ,<=, > .>=** 
- Ording 是Ord和PartialOrd的主要函数返回的枚举，描述了一种排序。
- Reberse 反转顺序的结构体
- max 、min 在Ord基础上建立的函数，允许找到两个值中的最大值和最小值



## 宏

Eq: 派生宏生成一个Eq特征

Ord：派生宏，生成特征Ord的impl

PartialEq: 派生宏，生成trait PartialEq的impl。

PartialOrd:  派生宏，生成trait PartialOrd的impl。

### 结构体

Reverse   用于反向排序的助手结构

### 枚举

Ordering：排序是两个值之间比较的结果。

### 特征

Eq: 等价关系的等式比较特征

Ord: 特征 用于构成总顺序的类型

PartialEq：部分等价关系的等式比较的特征。

PartialOrd： Trait可用于比较排序顺序的值。

### 方法

max： 比较并返回两个值的最大值。

min:   Compares and returns the minimum of two values.

max_by: [处于试验阶段] 返回关于指定比较函数中两个值的最大值

min_by: [处于试验阶段] 返回关于指定比较函数中两个值的最小值

max_by_key: [处于试验阶段]返回给定指定函数的最大值的元素。

min_by_key: [处于试验阶段]返回给定指定函数的最大值的元素。

#### max

```rust
 let y=78;
    let q=32;
    let max=cmp::max(q,y);
    println!("the y and q max value is {}",max);
```

#### min

```
 let y=78;
    let q=32;
    let min=cmp::min(q,y);
    println!("the y and q max value is {}",min);
```

