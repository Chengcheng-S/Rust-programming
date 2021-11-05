# std::assert_ne

```rust
macro_rules! assert_ne {
    ($left:expr, $right:expr) => { ... };
    ($left:expr, $right:expr,) => { ... };
    ($left:expr, $right:expr, $($arg:tt)+) => { ... };
}
```

断言两个表达式彼此**不相**等（使用PartialEq）

在panic时，此宏将打印表达式的值及其调试表示形式。

就像assert！，此宏可以在其中提供自定义的panic消息。

```rust
let a = 3;
let b = 2;
assert_ne!(a, b);

assert_ne!(a, b, "we are testing that the values are not equal");
```

