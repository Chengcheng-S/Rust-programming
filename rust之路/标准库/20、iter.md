# std::iter

可组合的外部迭代

模块组成： 本模块主要按类型组成

- traits 核心部分，这些特征定义了存在什么样的迭代器以及可以用它们做什么
- Functions  创建一些基本的迭代器
- Structs   该模块trait上各种方法的返回类型。

### Iterator

迭代器核心：

```rust
triat Iterator{
	type Item;
    fn next(&mut self)->Option<Self::Item>;
}
```

迭代器方法 `next` 当调用该方法时，返回一个Option<Item>,next 将返回Some<Item>,当元素迭代完之后返回`None`。 单个迭代器可能选择继续迭代，因此再次调用next可能最终也可能不会在某个时刻再次返回Some(Item)

迭代器也是可组合的，通常将它们链接在一起以执行更复杂的处理形式。

迭代器的完整定义还包括许多其他方法，但它们是默认方法，构建在next之上.

### 迭代三形式

- `iter()`   在`&T`基础上迭代
- `iter_mut()`  在`&mut T` 基础上迭代
- `into_iter()`  `T`的 基础上迭代

标准库中的各种东西可以实现这三种方法中的一种或多种。

### 实现迭代器

自定义迭代器步骤：

1. 创建一个结构以保持迭代器的状态
2. 为该结构实现迭代器

每个迭代器和迭代器适配器都有一个。

```rust
struct A{
    count:usize;
}
impl A{
    fn new()->A{
        A{count:0}
    }
}
impl Iterator for A{
    type Item=usize;
    fn next(&mut self)->Option<Self::item>{
        self.count+=1;
        if self.count<6{
            Some(self.count)
        }else{
            None
        }
    }
}
```

调用counter.next() 则会一次迭代Counter中的元素，直到None

### for 循环和intoIterator

Rust的for循环实际上是迭代器的语法糖

```rust
let valu=vec![1,2,3,4];
for x in valu{
    println!("{}",x);
}
```

所有迭代器通过返回自身来实现Iterator：

- 如果在写一个迭代器，可以把它和for循环一起使用。
- 如果正在创建一个集合，那么实现intointerator for将允许自定义集合与for循环一起使用。

### 适配器

**接收迭代器返回另一个迭代器的函数**通常称为“迭代器适配器”。

如果迭代器适配器死机，迭代器将处于未指定（但内存安全）状态。这种状态也不能保证在不同版本的Rust中保持相同， 避免依赖于迭代器返回的精确值，该迭代器会panic

### 惰性

迭代器(迭代器适配器)都是惰性的，即：仅创建一个迭代器并不能做什么，调用`next`之前什么也不会发生。

计算迭代器的另一种常见方法是使用collect方法生成新的集合。

### 无穷(Infinity)

迭代器不必是有界的

```rust
let a=0..;
```

通常使用`take Iterator`适配器将无限迭代器转为有限迭代器

```rust
let a=0..;
lett b=a.take(7);
for i in b{
	println!("{}",i);
}
```

注：在无限迭代器上的方法，即使是那些可以在有限时间内以数学方式确定结果的方法，也可能不会终止。

> 具体地说，像min这样的方法，在一般情况下需要遍历迭代器中的每个元素，对于任何无限迭代器，它可能不会成功返回。
