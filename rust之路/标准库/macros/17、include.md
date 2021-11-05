# std::include!

```rust
macro_rules! include{
	($file:expr)=>{...};
    ($file:expr,)=>{...};
}
```

根据上下文将文件解析为表达式或项

文件相对于当前文件定位（类似于如何找到模块）。提供的路径在编译时以特定于平台的方式进行解释。(使用包含反斜杠\的Windows路径的调用将无法正确编译。)

> 并不建议使用该宏，因为如果文件被解析为一个表达式，它将被不卫生地放在周围的代码中。如果当前文件中存在同名的变量或函数，这可能会导致变量或函数与文件预期的不同。

# std::include_bytes

```rust
mcaro_rules! include_bytes{
	 ($file:expr) => { ... };
     ($file:expr,) => { ... };
}
```

包含一个文件作为对字节数组的引用。

此宏将生成类型为&'static[u8；N]的表达式，该表达式是文件的内容

# std::include_str

```rust
macro_rules! include_str {
    ($file:expr) => { ... };
    ($file:expr,) => { ... };
}
```

以字符串形式包含UTF-8编码的文件

```rust
fn main() {
    let my_str = include_str!("spanish.in");
    assert_eq!(my_str, "adiós\n");
    print!("{}", my_str);
}
```

