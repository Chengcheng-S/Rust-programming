### std::backtrace::BacktraceStatus

```rust
#[non_exhaustive]
#[derive(Debug,PartialEq,Eq)]
pub enum BacktraceStatus{
	Unsupported, // 不支持回溯，可能因为当前平台未实现。 
    Disabled,  // 通过RUTS_LIB_BACKTRACE 或RUST_BACKTRACE环境变量已禁用捕获回溯
    Captured,  //已经捕获到回溯，并且Backtrace在渲染时应该返回合理的信息
}
```

回溯当前状态，指示它是被捕获还是 由于其他原因而为空。