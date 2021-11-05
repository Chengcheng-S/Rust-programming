# std::unimplemented!

```rust
macro_rules! unimplemented {
    () => { ... };
    ($($arg:tt)+) => { ... };
}
```

通过发出消息“未实现”来指示未实现的代码。

> 代码可以进行类型检查，这在原型或实现需要多种方法而不打算使用全部方法的特征时非常有用。

### panic

这将总是panic！因为unimplemented！只是panic!的速记带有固定的特定消息。就像panic！一样，此宏还有另一种形式用于显示自定义值