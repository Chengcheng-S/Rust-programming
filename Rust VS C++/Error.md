## Error Handing

### C++

C++允许代码抛出和捕获异常。抛出一个异常来中断当前的逻辑流，并允许堆栈上的某个对象捕获异常并恢复情况。如果没有任何东西捕捉到抛出，那么线程本身将退出。

```c++
void do_something() {
  if (!read_file()) {
    throw std::runtime_error("read_file didn't work!");
  }
}
...
try {
  do_something();
}
catch (std::exception e) {
   std::cout << "Caught exception -- " << e.what() << std::endl;
}
```

C++中的错误处理：

- 返回bool、int或具有特殊意义的指针的函数。如：false，-1或NULL表示失败。
- 返回结果代码或枚举的函数。
- 函数带有一个特殊的out参数。
- 提供`error()`中最后一个错误的详细信息的函数或者库提供类似的函数
- 捕获因任何失败而引发的异常。
- 返回错误的程序
- 重载分为 两种形式的函数，一种抛出异常，另一种将错误存储在错误参数中

### Rust

Rust提供了两种枚举类型Result和Option，允许函数将结果传播给调用方。

提供了`panic!()` 可用于代码中的意外状态和其他故障

#### Result<T,E>

```rust
enum Result<T,E>{
	Ok(T),
	Err(E)
}
```

在函数有异常时返回E，无异常时返回T。

### Option<T>

```rust
enum Option<T>{
	None
	Some(T)
}
```

要么返回None，要么就是T



















