### std::alloc::take_alloc_error_hook

```rust
pub fn take_alloc_error_hook()->fn(_:Layout)
```

此为先行版API

注销当前分配错误钩子，并返回

若未注册任何自定义钩子，则将返回默认钩子。

