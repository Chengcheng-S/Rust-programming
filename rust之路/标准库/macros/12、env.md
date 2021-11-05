# std::env

```rust
macro_rules! env{
	($name:expr)=>{...};
	($name:expr,)=>{...};
}
```

编译时检查环境变量

此宏将在编译时扩展到指定环境变量的值，从而生成一个类型为&'static str的表达式。

如果未定义环境变量，则会发出编译错误。要不发出编译错误，使用`opt。ion_env!`代替。

```rust
let path:&'static str=env!("PATH");
println!("the $PATH variable at the time of compiling was: {}", path);
```

 可以通过传递字符串作为**第二个参数自定义错误消息**

```rust
let doc:&'static str=env!("documentation", "what's that?!");
```

如果未指定，则会报错

```
error:what's that?
```

