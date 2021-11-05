# std::boxed

堆分配的指针类型

Box<T>  ==> Box  在Rust中提供了最简单的对分配形式。 提供此分配的所有权，并在其超出范围时删除其内容， 还确保它们分配的字节数不会超过`isize::MAX` 字节。

### 创建递归结构

```rust
#[derive(Debug)]
enum List<T>{
	Cons(T,Box<List<T>>),
	Nil,
}
let list:List<i32>=List::Cons(1,Box<List::Cons(2,Box::new<List::Nil>));
println!("{:?}",list);  // Cons(1,Cons(2,Nil))
```

递归的结构必须 Box，若`Cons` 的定义如下：

```rust
Cons(T,List<T>);
```

这将无法工作，这是因为列表的大小取决于列表中有多少元素，因此不知道要为Cons分配多少内存。通过引入一个Box<T>，它有一个定义的大小，

### 内存布局

对于非零大小的值，Box将使用全局分配器进行分配。在使用全局分配器分配的Box和原始指针之间进行双向转换是有效的，前提是分配器使用的布局对于类型是正确的。