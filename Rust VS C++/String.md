## string

C/C++没有字符串基元类型，而是具有char类型，即一个字节。`string`是指向以零字节`\0`结尾的字符数组的指针。

```c++
// The array that my_string points at ends with a hidden \0
char *my_string = "This is as close to a string primitive as you can get";
```

- C中`strlen()`,`strcpy()`,`strdup()` 等函数允许对字符串进行操作，但他们通过使用零字节来确定事务的长度来工作。因此`strlen()` 原理则是找到`\0`之前遇到的字节数。**在这些函数找到终止字符之前，他们很容易意外的溢出缓冲区**。
- C++ 中 `std::string`包装了char指针，并提供了以安全方式修改字符串的安全方法。弥补了C中的不足，但该方法是在标头中定义的类，与其他的类一样，被编译并链接到可执行文件。
- 此外，`std::sting`通常会使用堆来存储字符串数据(这么做，对内存的使用是十分不利的)将一个字符串分配给另一个字符串会产生隐性成本，因为必须分配内存才能接收字符串的副本，即使在分配过程中未修改字符串本身也是如此。

C/C++中通过创建`wchar_t`对Unicode的支持，C++中也有等效的方法`std::wstring`。

标准C ++库还具有`std :: basic_string`，可充当各种字符类型的包装，并可用于处理任何宽度的字符串。专门用作`std :: string`，`std：wstring`，`std :: u16string`和`std :: u32string`。

> 因为wchar_t类型可以为2或4个字节宽，并且是编译器/平台特定的决定。在Microsoft Visual C ++中，宽字符是无符号的短字符（对应于Win32的Unicode API），在gcc中，根据编译标志，它可以是32位或16位。 16位值将包含基本多语言平面中的符号，但不包含完整的32位范围。这意味着应将16位宽的字符串假定为UTF-16编码，因为否则它们将无法正确支持Unicode。 C ++ 11通过引入显式的char16_t和char32_t类型以及称为std :: u16string和std :: u32string的字符串的相应版本来纠正此问题。

C++中的四种字符类型：

| type     | Encoding             |
| -------- | -------------------- |
| char     | C,ASCII.EBDIC,UTF-8, |
| wchar_t  | UTF-16 UTF-32        |
| char16_t | UTF-16               |
| char32_t | UTF-32               |

### Rust

- 字符类型是32位的Unicode字符，足以容纳单个字符 
- `str`类型是保存在内存中的UTF-8编码的字符串。实际中经常使用`&str`这是字符串的切片，一个str不需要以零字节结尾，并且在实际生产中可以包含零字节。
- `std::String`  堆分配字符串类型，构建字符串，读取，克隆。
- `OSString` 特定于平台的类型，它可以处理操作系统如何看待字符串的任何差异。在Windows上，它将返回UTF-16字符串，在Linux / Unix系统上，它将返回UTF-8。 OsStr是OsString上的一个切片，类似于str和String。

**注**：

- 内部使用UTF-8进行编码，但char是32位。字符串的长度被认为是其字节长度。有一些特殊的迭代器可以遍历字符串并将UTF-8解码为32位字符。
- str值内存中某个字节的不可变数组，当str指向String时，str可能在堆上，若String是静态的，则str可能在全局内存之中。

直观对比：

| C++                                | Rust          |
| :--------------------------------- | :------------ |
| `char *` or `wchar_t *`            |               |
| C++11 - `char16_t *`, `char32_t *` | `str`, `&str` |
| `std::string`, `std::wstring`      |               |
| `std::u16string` `std::u32string`  | `std::String` |

### Display 和Debug trait

Rust中允许格式化字符串，若要输出需要实现以下两个trait中的任意一个：

- `Display`——用于类型的标准文本表示。
- `Debug`——用于调试类型的文本表示形式。

对象实现Display

```rust
use std::fmt::{Display,Formatter};

strutc Person{
	Name String
}

impl Display for Person{
	fn fmt(&slef,f:&mut Formatter)->fmt::Result{
        write!(f,"{} is his name!!!",self.Name)
    }
}
```

实现Debug

```rust
use std::fmt::{Display,Formatter};

#[derive(Debug)]
strutc Person{
	Name String
}

```

### 格式化字符串

| C++                    | Rust           |                                                              |
| ---------------------- | -------------- | ------------------------------------------------------------ |
| %s、%u、%d、%i、%f、%c | {}             | C / C ++要求指定参数的类型。在Rust中，可以推断类型，{}会调用该类型的Display trait |
| %lld 、%llu            | {}             | C / C ++具有扩展功能以处理不同大小的int和float，Rust中不需要这么做 |
|                        | {:?}  {:#?}    | 在Rust中{:?} 返回由类型的Debug特性实现的所有内容。 而{:#?}则是格式化输出的结构 |
| %-10s                  | {:<10}         | 设置左对齐字符串的格式，至少填充10个空格                     |
| `%04`                  | `{:04}`        | 用零填充一个宽度为4的数字                                    |
| `%.3`                  | `{:.3}`        | 将数字的精度填入小数点后三位。也可以是零填充的，例如{.03}    |
| `%e`, `%E`             | `{:e}`, `{:E}` | 小写或大写指数                                               |
| `%x`, `%X`             | `{:x}`, `{:X}` | 小写或大写的十六进制。注意{：#x}，{：#x}以0x作为输出的前缀   |
| `%o`                   | `{:o}`         | 八进制。注意{：#o}在输出的前面加上0o                         |
|                        | {:b}           | 二进制 注意{：#b}在输出的前面加上0b                          |
| `%p`                   | `{:p}`         | 指针                                                         |

