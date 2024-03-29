# std::result

`result`类型的错误处理

`Result<T,E>`  用于返回和传播错误的类型，它是一个包含变量Ok(T)的**枚举**，表示成功并包含一个值，Err(E)表示错误并包含返回值。

```rust
enum Result<T,E>{
	Ok(T),
	Err(E),
}
```

函数在预期错误和可恢复错误时返回结果。在std库中，Result最常用于I/O。

### used

使用返回值指示错误的一个常见问题是，很容易忽略返回值，从而无法处理错误。

Result 必须使用`#[must_use]`属性进行注释，将导致编译器在忽略Result时发出警告，使得Result对于可能遇到错误但是不会返回有用值的函数特别有用。

### ？运算符

当编写调用许多返回结果类型的函数的代码时，错误处理可能是乏味的。问号运算符？，隐藏了一些向调用堆栈传播错误的样板文件。

? 只能在返回Result的函数中使用，因为它提供了Err的早期返回。

### structs

| IntoIter | Result的Ok比那辆中值的迭代器 |
| -------- | ---------------------------- |
| Iter     | Result中Ok引用的迭代器       |
| IterMut  | Result中Ok可变引用的迭代器   |

### Enums

Result   	 表示成功（Ok）或失败（Err）的类型

