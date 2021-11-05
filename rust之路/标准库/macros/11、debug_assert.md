# std::debug_assert

```rust
macro_rules! debug_assert{
	($($arg:tt)*)=>{...};
}
```

断言布尔表达式在运行时为真。

### uses

> 不像assert！，debug_assert！默认情况下，只有在未优化的生成中才启用语句。优化后的生成不会执行debug_assert！语句，除非将-C debug_assert传递给编译器。这将使debug_assert！对于成本太高而无法在发布版本中出现的检查非常有用，但在开发过程中可能会有所帮助。扩展debug_assert的结果！总是进行类型检查。

未经检查的断言允许处于不一致状态的程序继续运行，这可能会产生意外的后果，但不会引入不安全，只要这只发生在安全代码中。然而，断言的性能成本通常是不可测量的。替换assert！用debug_assert！  只在安全代码中使用。

# debug_assert_eq!

```rust
macro_rules! debug_assert_eq {
    ($($arg:tt)*) => { ... };
}
```

断言两个表达式彼此相等。

panic时，此宏将打印表达式的值及其调试表示形式

# debug_assert_ne!

```rust
macro_rules! debug_assert_ne {
    ($($arg:tt)*) => { ... };
}
```

断言两个表达式不相等，

panic时，此宏将打印表达式的值及其调试表示形式

```rust
let a = 3;
let b = 2;
debug_assert_ne!(a, b);
```