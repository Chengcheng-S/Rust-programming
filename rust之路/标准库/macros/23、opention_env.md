# std::option_env!

```
macro_rules! option_env {
    ($name:expr) => { ... };
    ($name:expr,) => { ... };
}
```

可选地在编译时检查环境变量

如果指定的环境变量在编译时存在，则这将扩展为Option<&'static str>类型的表达式，其值是环境变量的一些值。如果环境变量不存在，则该变量将扩展为无。

对于此宏 ，无论环境变量是否存在，都不会发出编译时错误。

```rust
let key: Option<&'static str> = option_env!("SECRET_KEY");
println!("the secret key might be: {:?}", key);
```

