# std::compile_error

```rust
mcaro_rules! compile_error{
	($msg:expr)=>{...};
	($msg:expr,)=>{...};
}
```

导致编译在遇到给定错误消息时失败。

当crates使用条件编译策略为错误条件提供更好的错误消息时，应使用此宏。这是**编译器级的panic！**，但在**编译期间**而不是在运行时发出错误。

### eg

```rust
macro_rules a{
	(foo)=>{};
	(bar)=>{};
	($x:ident)=>{
		compile_error!("this macro only accepts foo or bar");
	}
}
a(another);
```

如果许多功能之一不可用，则发出编译器错误。

```rust
#[cfg(not(any(feature = "foo", feature = "bar")))]
compile_error!("Either feature \"foo\" or \"bar\" must be enabled for this crate.");
```

