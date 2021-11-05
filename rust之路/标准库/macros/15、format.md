# std::format!

```rust
macro_rulse! format{
	($($arg:tt)*)=>{};
}
```

使用运行时表达式的插值创建字符串

第一个参数 receives是一个格式字符串。这必须是字符串文本。格式化字符串的强大功能在{}中。

传递到格式的其他参数！除非使用命名参数或位置参数，否则按给定的顺序替换格式化字符串中的{}。

### panic

如果格式化特征实现返回错误，则出现panic。

```rust
format!("test");
format!("hello {}", "world!");
format!("x = {}, y = {y}", 10, y = 30);
```

# std::format_args

```rust
macro_rules! format_args{
	($fmt:expr)=>{...};
	($fmt:expr,$($args:tt)*)=>{...};
}
```

为其他字符串格式宏构造参数。

这个宏的作用是为每个传递的额外参数获取一个包含{}的格式化字符串文本。

此宏生成fmt:：Arguments类型的值。此值可以传递给std:：fmt中的宏，以执行有用的重定向。

```rust
let debug = format!("{:?}", format_args!("{} foo {:?}", 1, 2));
let display = format!("{}", format_args!("{} foo {:?}", 1, 2));
assert_eq!("1 foo 2", display);
assert_eq!(display, debug);
```

# std::format_nl!

```rust
macro_rules! format_args_nl {
    ($fmt:expr) => { ... };
    ($fmt:expr, $($args:tt)*) => { ... };
}
```

先行版 API   类似于`format_args`  之后多一个换行符









