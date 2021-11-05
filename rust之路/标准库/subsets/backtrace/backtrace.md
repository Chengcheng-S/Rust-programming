# std::backtrace

先行版的API

支持捕获OS线程的堆栈回溯。

此模块包含捕获的堆栈回溯所必须的支持，从OS线程本身运行OS线程， `Backtrace`类型支持， 通过`Backtrace::capture`和`Backtrace::force_capture` 捕获堆栈跟踪。

回溯容易被附加错误(类型实现了std::error::Error)。 以获取产生错误的原因 。

### 环境变量

`Backtrace::capture` 实际上不会通过捕获回溯，其由两个环境变量控制

- `RUST_LIB_BACKTRACE`  若设置为0，则 `Backtrace::capture` 永远不会捕获回溯。
- `RUST_BACKTRACE` 如果未设置`RUST_LIB_BACKTRACE`，
- 均未设置时，`Backtrace::capture` 将被禁用。

注：`Backtrace :: force_capture`函数可以用来忽略1这些环境变量。一旦创建了第一个回溯，变量就被缓存，所以改变在运行时，`RUST_LIB_BACKTRACE`或`RUST_BACKTRACE`可能实际上不会更改。





