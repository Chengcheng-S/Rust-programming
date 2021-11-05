# std::cfg

```rust
macro_rules! cfg{
	($($cfg::tt)*)=>{...};
}
```

编译时计算配置标志的布尔组合

除`#[cfg]` 属性外， 还提供此宏以允许布尔表达式计算配置标志。这通常会导致较少的重复代码。

给这个宏的语法与cfg属性的语法相同。

cfg！与#[cfg]不同，它不删除任何代码，只计算为true或false。

```rust
let my_directory = if cfg!(windows) {
    "windows-specific-directory"
} else {
    "unix-directory"
};
```

当cfg！时，if/else表达式中的所有块都必须是有效的！

