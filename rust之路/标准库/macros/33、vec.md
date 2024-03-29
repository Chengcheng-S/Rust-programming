# std::vec!

```rust
macro_rules! vec {
    () => { ... };
    ($elem:expr; $n:expr) => { ... };
    ($($x:expr),+ $(,)?) => { ... };
}
```

创建`Vec` 

允许使用与数组表达式相同的语法定义vec。此宏有两种形式：

- 创建包含给定元素列表的Vec：

  ```rust
  let v = vec![1, 2, 3];
  ```

- 从给定元素和大小创建Vec：

  ```rust
  let v = vec![1; 3];
  ```

请注意，**与数组表达式不同，此语法支持实现Clone的所有元素，并且元素的数量不必是常量。**

这将使用clone来复制表达式，因此对于具有非标准克隆实现的类型，应该小心使用它。



