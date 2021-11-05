# std::primitive

重新导出基元类型，以运行其他声明类型不可能隐藏得用法，这通常只在宏生成得代码中有用

bool 是一个struct而不是基元类型，需要重导出

```rust
pub struct bool;

impl QueryId for bool {
    const SOME_PROPERTY: core::primitive::bool = true;
}
```

