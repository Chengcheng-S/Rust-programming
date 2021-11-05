# alloc::boxed

用于堆分配的指针类型。

Box<T>通常被称为“Box”，它在Rust中提供了最简单的堆分配形式。Box提供此分配的所有权，并在超出范围时删除其内容。Box还确保它们分配的字节数永远不会超过isize:：MAX bytes。

通过创建一个Box将值从堆栈移动到堆:

```rust
let val: u8 = 5;
let boxed: Box<u8> = Box::new(val);
```

通过解引用将值从Box移回堆栈：

```rust
let boxed: Box<u8> = Box::new(5);
let val: u8 = *boxed;
```

创建递归数据结构

```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}

let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
```

### 内存布局

对于非零大小的值，box将使用全局分配器进行分。在使用全局分配器分配的box和原始指针之间**进行双向转换是有效**的，前提是分配器使用的布局对于类型是正确的。

> 只要T:Sized  一个Box<T> 就可以保证表示为单个指针，并且与C指针兼容。 如果有从C调用的外部“C”Rust函数，则可以使用Box<T>类型定义这些Rust函数，并在C端使用T*作为相应的类型。















