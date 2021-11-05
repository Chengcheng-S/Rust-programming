# std::panic!

```rust
macro_rules! panic {
    () => { ... };
    ($msg:expr) => { ... };
    ($msg:expr,) => { ... };
    ($fmt:expr, $($arg:tt)+) => { ... };
}
```

使当前线程panic

这允许程序立即终止并向程序的调用者提供反馈。panic！应在程序达到不可恢复状态时使用。

此宏是在示例代码和测试中断言条件的最佳方法。panic！与Option和Result枚举的unwrap方法密切相关。两种实现都叫恐慌！当它们被设置为None或Err变量时。

此宏用于向Rust线程注入panic，导致线程完全死机。每个线程的panic可以作为Box<Any>类型和panic的单参数形式将是传输的值。

对于从错误中恢复，Result enum通常比使用panic!更好

如果主线程死机，它将终止所有线程，并以代码101结束程序。

