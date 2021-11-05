# std::line

```rust
macro_rules! line{
		()=>{};
}
```

扩展到调用它的行号

> 扩展表达式的类型为u32，基于1，因此每个文件中的第一行计算为1，第二行计算为2，依此类推。这与常见编译器或流行编辑器的错误消息一致。

```rust
let current_line = line!();
println!("defined on line: {}", current_line);
```

