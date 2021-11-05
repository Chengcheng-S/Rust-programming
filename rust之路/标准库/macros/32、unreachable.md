# std::unreachable!

```rust
macro_rules! unreachable {
    () => { ... };
    ($msg:expr) => { ... };
    ($msg:expr,) => { ... };
    ($fmt:expr, $($arg:tt)*) => { ... };
}
```

指示无法访问的代码

当编译器无法确定某些代码无法访问时，这将非常有用

- 守卫匹配
- 动态终止的循环
- 动态终止的迭代器。

如果代码不可访问的判断被证明是不正确的，程序将立即终止并发出一个panic!

此宏的不安全对应项是unreachable_unchecked函数，如果到达代码，它将导致未定义的行为。

```rust
fn foo(x: Option<i32>) {
    match x {
        Some(n) if n >= 0 => println!("Some(Non-negative)"),
        Some(n) if n <  0 => println!("Some(Negative)"),
        Some(_)           => unreachable!(), // compile error if commented out
        None              => println!("None")
    }
}
```

迭代

```rust
fn divide_by_three(x: u32) -> u32 { // one of the poorest implementations of x/3
    for i in 0.. {
        if 3*i < i { panic!("u32 overflow"); }
        if x < 3*i { return i-1; }
    }
    unreachable!();
}
```

