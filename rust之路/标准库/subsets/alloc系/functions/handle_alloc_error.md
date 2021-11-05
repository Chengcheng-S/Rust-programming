### std::alloc::handle_alloc_error

```rust
pub fn handle_alloc_error(layout:Layout)->|
```

中止内存分配错误或失败

对于那些希望响应分配错误而中止计算的内存分配API调用者，推荐使用此函数，并非直接panic。默认行为是将消息打印为 `stderror`并终止该过程，可以替换为`set_alloc_error_hook`和`take_alloc_error_hook`。

