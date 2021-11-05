# std::module_path!

```rust
macro_rules! module_path {
    () => { ... };
}
```

展开为表示当前模块路径的字符串。



当前的模块路径可以看作是模块的层次结构，这些模块通向板条箱根目录。返回的路径的第一个组件是当前正在编译的crates的名称。

```
mod test {
    pub fn foo() {
        assert!(module_path!().ends_with("test"));
    }
}

test::foo();
```

