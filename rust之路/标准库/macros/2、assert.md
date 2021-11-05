# std::assert

```rust
macro_rules! assert{
	($cond:expr)=>{...};
	($cond:expr,) => { ... };
    ($cond:expr, $($arg:tt)+) => { ... };
}
```

断言布尔表达式在运行时为真。

若表达式为false  则panic

### 用途

断言总是在调试和发布版本中检查，并且不能被禁用。查看debug_assert！对于默认情况下在发布版本中未启用的断言。

不安全代码可能依赖于assert！强制执行运行时不变量，如果违反这些不变量可能会导致不安全。

在安全代码中包括测试和强制执行运行时不变量（其违反不会导致不安全）。

### 自定义消息

这个宏有第二种形式，可以提供自定义的紧急消息，也可以不带格式参数。

```rust
let x = true;
assert!(x, "x wasn't true!");

let a = 3; let b = 27;
assert!(a + b == 30, "a = {}, b = {}", a, b);
```

