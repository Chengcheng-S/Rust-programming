# std::stringify!

```rust
macro_rules! stringify {
    ($($t:tt)*) => { ... };
}
```

使其参数字符串化

这个宏将产生一个类型为&'static str的表达式，它是传递给宏的所有标记的字符串化。宏调用本身的语法没有任何限制。

```rust
let one_plus_one = stringify!(1 + 1);
assert_eq!(one_plus_one, "1 + 1");
```

