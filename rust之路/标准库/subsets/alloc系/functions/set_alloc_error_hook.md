### std::alloc::set_alloc_error_hook

```rust
pub fn set_alloc_error_hook(hook:fn(_:Layout))
```

此为先行版的API

注册一个自定义分配错误的钩子，替换已注册的任何钩子

当可靠的内存分配失败时(在运行终止之前)，将调用分配错误的钩子，默认的钩子将向stderr打印一条信息，但是可以使用`set_alloc_error_hook`和`take_alloc_error_hook`函数自定义此项行为，

该钩子提供了一个Layout结构，该项结构包含有关失败分配的信息，分配错误钩子是全局资源。

