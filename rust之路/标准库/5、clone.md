## clone

### std::clone

无法隐式复制的类型的`Clone`trait,

在Rust一些简单类型是“隐式可复制的”，当将其赋值或者作为参数传递时，接收方将接受一个副本，保留原始值。这些类型不需要分配来复制，也没有终结器（即，它们不包含拥有的框或实现Drop）因此编译器认为复制它们即便宜又安全，

对于其他的类型，必须显式地复制，方法则是：实现Clone triat并调用Clone方法

```rust
let s = String::new(); // String type implements Clone
let copy = s.clone(); // so we can clone it
```

实现Clone triat

```rust
#[derive(Clone)] 
struct Morpheus {
   blue_pill: f32,
   red_pill: i64,
}

fn main() {
   let f = Morpheus { blue_pill: 0.0, red_pill: 0 };
   let copy = f.clone(); // and now we can clone it!
}
```