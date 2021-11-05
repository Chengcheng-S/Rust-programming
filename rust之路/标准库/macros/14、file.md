# std::file

```rust
macro_rules! file{
	()=>{};
}
```

扩展到调用它的文件名

用`cloumn!`还有`line!`，这些宏为开发人员提供有关源中位置的调试信息。

扩展表达式的类型是`&static str`，返回的文件不是对该文件的`file!`本身，而是导致文件调用的第一个宏。



