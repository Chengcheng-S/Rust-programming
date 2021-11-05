# std::write!

```rust
macro_rules! write {
    ($dst:expr, $($arg:tt)*) => { ... };
}
```

将格式化数据写入缓冲区

此宏接受格式字符串、参数列表和“writer”。参数将根据指定的格式字符串格式化，结果将传递给编写器。

一个模块可以同时导入std:：fmt:：Write和std:：io:：Write并调用Write！在实现其中一个的对象上，因为对象通常不会同时实现这两者。但是，模块必须导入限定的特征，以便它们的名称不冲突：

注：此宏也可用于no_std。在无标准设置中，负责组件的实现细节。

```rust
use core::fmt::Write;

struct Example;

impl Write for Example {
    fn write_str(&mut self, _s: &str) -> core::fmt::Result {
         unimplemented!();
    }
}

let mut m = Example{};
write!(&mut m, "Hello World").expect("Not written");
```

# std::writeln!

```rust
macro_rules! writeln {
    ($dst:expr) => { ... };
    ($dst:expr,) => { ... };
    ($dst:expr, $($arg:tt)*) => { ... };
}
```

将格式化数据写入缓冲区，并附加一个换行符。

