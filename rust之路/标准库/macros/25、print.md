# std::print!

```rust
macro_rules! print {
    ($($arg:tt)*) => { ... };
}
```

打印到标准输出

相当于println！宏，但在消息末尾不打印换行符。

使用print! 只用于程序的主要输出。使用eprint！而是打印错误和进度消息。

# std::println!

类似于print!   比它多一个 换行符

```rust
macro_rules! println {
    () => { ... };
    ($($arg:tt)*) => { ... };
}
```

