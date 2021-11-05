# std::eprint

```rust
macro_rules! eprint{
	($($arg:tt)*)=>{...};
}
```

打印到标准错误

相当于`print!` 但是输出到io::stderr,而不是io::stdout 

使用eprint！仅用于错误和进度消息。

### panics

写入 `io::stderr` 时panic

```
eprint!("Error: Could not complete task");
```

# std::eprintln！

```rust
macro_rules! eprintln {
    () => { ... };
    ($($arg:tt)*) => { ... };
}
```

比eprint多一个换行
