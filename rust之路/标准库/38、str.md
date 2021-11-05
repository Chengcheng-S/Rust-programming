# std::str

Unicode 字符串切片

&str 是Rust中两种主要的字符串类型之一，另一种为String，不同于字符串，它的内容是借用的。

### 基本用法

str类型的基本字符串声明

```rust
let hello_world: &'static str = "Hello, world!";
```

### Modules

pattern                       实验： 字符串模式API

### Traits 

FromStr					从字符串分析值

### Functions

| from_boxed_utf8_unckecked | 将box字节片转换为box字符串片，而不检查字符串是否包含有效的UTF-8。 |
| ------------------------- | ------------------------------------------------------------ |
| from_utf8                 | 将字节切片转换为字符串切片                                   |
| from_utf8_mut             | 将字节切片转换为可变的字符串切片                             |
| from_utf8_mut_unchecked   | 在不检查字符串是否包含有效的UTF-8的情况下，将字节切片转换为字符串切片。 |
| from_utf8_unchecked_mut   | 将字节片转换为字符串切片，而不检查字符串是否包含有效的UTF-8；可变版本。 |

# std::string

UTF-8编码的可增长字符串。

此模块包含字符串类型、一个用于转换`ToString`的特征，以及使用字符串可能导致的几种错误类型。

创建通过使用`+`

```rust
let s = "Hello".to_string();

let message = s + " world!";
```

### Traits

ToString									用于将值转换为字符串

### Type Definition

ParseError						`Infallible`别名

### Strutcs

| Drain          | 字符串的耗尽迭代器                             |
| -------------- | ---------------------------------------------- |
| FromUtf8Error  | 从UTF-8字节向量转换字符串时可能出现的错误值。  |
| FromUtf16Error | 从UTF-16字节向量转换字符串时可能出现的错误值。 |
| String         | UTF-8编码的可增长字符串。                      |

