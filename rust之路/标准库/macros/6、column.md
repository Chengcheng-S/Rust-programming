# std::column

```rust
macro_rules! column{
	()=>{};
}
```

扩展到调用它的列号

用`line!`还有`file!`，这些宏为开发人员提供有关源中位置的调试信息。

```rust
let current_col = column!();
println!("defined on column: {}", current_col);
```

扩展表达式的类型为u32，基于1，因此每行中的第一列计算为1，第二列的计算结果为2，依此类推。这与常见编译器或流行编辑器的错误消息一致。返回的列不一定是列的column！调用本身，而是导致调用列的第一个宏调用column!。