# std::Option

可选值

Option 表示一个可选值，每个选项要么是`Some(value)`,要么是`None`.

用途：

- 初始值
- 未在整个输入范围内定义的函数的返回值(部分函数)
- 用于报告简单错误的返回值，其中出错时不返回任何错误
- 可选的结构字段
- 可借出或“获取”的结构字段
- 可选函数参数
- 空指针
- 把事情从困境中解脱出来

Option常与模式匹配结合使用，以查询值得存在并采取行动，始终考虑None得情况。

### Options 和指针(空指针)

Rust得指针类型必须始终指向有效位置，没有`null`引用。相反，Rust有Option指针，比如：·Option<Box<i32>>

```rust
let optional = None;
check_optional(optional);

let optional = Some(Box::new(9000));
check_optional(optional);

fn check_optional(optional: Option<Box<i32>>) {
    match optional {
        Some(p) => println!("has value {}", p),
        None => println!("has no value"),
    }
}
```

### Structs

| IntoIter  | Some得某个变量上得值得迭代器                    |
| --------- | ----------------------------------------------- |
| Iter      | 对Some的某个变量的引用的迭代器。                |
| IterMut   | 对Some得可变引用迭代器                          |
| NoneError | 应用try运算符（？）产生的错误类型设置为None值。 |

















