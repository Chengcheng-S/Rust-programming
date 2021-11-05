# std::panic

标准库中的panic支持

### Structs

| AssertUnwindSafe | 简单的装饰器，用于断言 类型安全 |
| ---------------- | ------------------------------- |
| Location         | 一个包含有panic位置信息的结构。 |
| PanicInfo        | 提供有关结构的panic信息。       |

### Traits

| RefUnwindSafe | 在表示引用的共享trait被认为是安全的。 |
| ------------- | ------------------------------------- |
| UnwindSafe    | 代表Rust panic安全类型的标记trait     |

### Functions

| catch_unwind  | 调用一个闭包，捕获在发生展开panic时引起的原因     |
| ------------- | ------------------------------------------------- |
| resume_unwind | 在不调用panic 钩子函数的情况下 触发panic          |
| set_hook      | 注册一个自定义的panic钩子，替换以前注册的任何一个 |
| take_hook     | 注销当前的panic 钩子，并返回                      |