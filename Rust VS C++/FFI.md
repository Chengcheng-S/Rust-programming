## Foreign Function Interface

Rust 有一个外部函数接口，可以定义外部函数，公开自己的函数，并调用不安全的代码，否则这些代码在Rust中是非法的。

### 调用C函数

Rust支持外部函数接口的概念，它是在链接时解析的外部函数或类型的定义。

```rust
#[link(name="foo")]
extern{
	fn foo_command(command:*mut u8)
}
```

调用此函数，需要在`unsafe`块中：

```rust
pub fn run_command(command:&[u8]){
	unsafe{
		for_command(command.as_ptr());
	}
}
```

Rust中专门提供了一个`libc` 的第三方库，包含了c中的函数

需要在`cargo.toml`中进行设置

```rust
[dependencies]
libc="0.2.17"
```



```rust
extern crate libc;
use libc::{c_char,malloc,free,atol};
```

