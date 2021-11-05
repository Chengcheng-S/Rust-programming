# std::concat

```rustr
macro_rules! concat {
    ($($e:expr),*) => { ... };
    ($($e:expr,)*) => { ... };
}
```

将文本连接到静态字符串切片中

此宏接受任意数量的逗号分隔的文本，生成一个类型为&'static str的表达式，该表达式表示从左到右连接的所有文本。整数和浮点字面值被串接起来。

```rust
let x= concat!("test",10,"b",true);
assert_eq!(x, "test10btrue");
```

