# std::try!

```rust
macro_rules! try {
    ($expr:expr) => { ... };
    ($expr:expr,) => { ... };
}
```

在Rust 1.39.0 之后使用`?` 代替

展开结果或传播其错误。

