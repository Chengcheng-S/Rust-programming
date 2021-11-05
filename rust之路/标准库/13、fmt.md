# std::fmt

格式化输出Strings

此模块包含对格式的运行时支持！语法扩展。此宏在编译器中实现，以发出对此模块的调用，以便在运行时将参数格式化为字符串。

## 使用

`format!` 宏 是为了让那些来自C的printf/fprintf函数或Python的str.格式功能。

```rust
format!("Hello");                 // => "Hello"
format!("Hello, {}!", "world");   // => "Hello, world!"
format!("The number is {}", 1);   // => "The number is 1"
format!("{:?}", (3, 4));          // => "(3, 4)"
format!("{value}", value=4);      // => "4"
format!("{} {}", 1, 2);           // => "1 2"
format!("{:04}", 42);             // => "0042" with leading zeros
```

将单个值转换为字符串`to_string`

### 位置参数

每个格式参数都可以指定它引用的值参数，如果省略，则假定它是“下一个参数”。

> 例如，格式字符串{}{}{}}{}将采用三个参数，并且它们的格式顺序与给定的顺序相同。但是，格式字符串{2}{1}{0}将以相反的顺序格式化参数。

```rust
format!("{1} {} {0} {}",1,2);====>"2 1 1 2"
```

格式字符串需要使用它的所有参数，否则是编译时错误。在格式字符串中可以多次引用同一参数。

### 命名参数

Rust本身没有一个类似Python的函数命名参数，而是它的format！宏是一个语法扩展，允许它利用命名参数。参数的语法和参数在列表末尾列出：

```rust
format!("{argument}", argument = "test");   // => "test"
format!("{name} {}", 1, name = 2);          // => "2 1"
format!("{a} {c} {b}", a="a", b='b', c=3);  // => "a 3 b"
```

**将位置参数（没有名称的参数）放在具有名称的参数之后是无效的。与位置参数一样，提供格式字符串未使用的命名参数是无效的。**

### 格式化参数

每个被格式化的参数都可以通过一些格式化参数（对应于语法中的format_spec）进行转换。这些参数影响正在格式化的字符串表示。

```rust
// All of these print "Hello x    !"
println!("Hello {:5}!", "x");
println!("Hello {:1$}!", "x", 5);
println!("Hello {1:0$}!", 5, "x");
println!("Hello {:width$}!", "x", width = 5);
```

格式应采用的“最小宽度”参数。如果值的字符串没有填满这么多字符，那么fill/alignment指定的填充将用于占用所需的空间。

**宽度值也可以在参数列表中作为usize提供，方法是添加后缀$，指示第二个参数是指定宽度的usize**。

*使用dollar语法引用参数不会影响“next argument”计数器，因此通常最好按位置引用参数或使用命名参数。*

### 填充/对齐

可选的填充字符和对齐方式通常与宽度参数一起提供。它必须在width之前定义，紧跟在`:`之后。这表示如果正在格式化的值小于宽度，将在其周围打印一些额外的字符。对于不同的路线，填充有以下变体：

- `[fill]<`  参数在宽度列中左对齐
- `[fill]^` 参数在宽度列中居中对齐
- `[fill]>` 右对齐

非数字的默认填充/对齐方式是空格和左对齐。数字格式设置程序的默认值也是空格字符，但对齐方式正确。如果为数字指定了0标志，则隐式填充字符为0。

注：某些类型可能无法实现对齐，尤其是 通常不是针对Debug trait实现的。确保应用填充的一个好方法是格式化输入，然后填充此结果字符串以获得输出

```rust
println!("Hello {:^15}!", format!("{:?}", Some("hi"))); // => "Hello   Some("hi")   !"
Run
```

### 签名

格式化程序：

- `+`   用于数字类型，并指示应始终打印符号。默认情况下，不会打印+号，只有在默认情况下，才会为有符号trait打印负号，此标志表示应始终打印正确的符号（+或-）
- `-` 当前未使用。
- `#`  使用“alternate”形式打印， alternate形式：
  - `#?`  Debug format
  - `#x`  参数前加一个 `0x`
  - `#X`  参数前加一个`0X`
  - `#b`  参数前加一个`0b`
  - `#o`  参数前加一个`0o`
- `0`  指示对于整数格式，宽度的填充既要使用0字符，也要有符号意识。像{:08}这样的格式将为整数1生成00000001，而相同的格式将为整数-1生成-0000001。
  - 注： 负比正少一个零
  - 填充零总是放在符号（如果有）之后和数字之前。当与#标志一起使用时，会应用类似的规则：在前缀后面但在数字之前插入填充零。前缀包含在总宽度中。

### 精度

对于非数字类型，这可以视为“最大宽度”。如果得到的字符串比这个宽度长，那么它将被截短到这么多个字符，并且如果设置了这些参数，那么截断的值将以适当的填充、对齐和宽度发出。

- 对于整数类型，将被忽略
- 对于浮点类型，表示小数点后的位数

指定所需精度的方法：

- 整数 `.N` N 本身就是精度
- `N$`  使用格式参数N 必须是**usize**
- `.*`.*表示此{…}与两个格式输入关联，而不是一个：第一个输入保存usize精度，第二个输入保存要打印的值。 
  - 在这种情况下，如果使用格式字符串{<arg>：<spec>*}，那么<arg>部分引用要打印的值，精度必须在<arg>之前的输入中。

**Rust的标准库提供的格式函数没有任何语言环境的概念，无论用户配置如何，在所有系统上都会产生相同的结果。**

### Format traits

当请求用特定类型格式化参数时，实际上是在请求将参数归因于特定的特征。

当前类型到特征的映射：

- nothing => `Display`
- `?`=> `Debug`
- `x?`=> `Debug`使用小写的十六进制整数
- `X?`=> `Debug` 使用大写的十六进制整数
- `o`=> `Octal`
- `x`=>`LowerHex`
- `X`=>`UpperHex`
- `p`=> `Pointer`
- `b`=>`Binary`
- `e`=>`LowerExp`
- `E`=>`UpperExp`

这意味着实现fmt:：Binary特征的任何类型的参数都可以用{:b}格式化。

### fmt::Display  VS fmt::Debug

- fmt::Display  实现断言，类型可以始终表现为`utf-8`字符串，并非所有类型都实现该trait
- fmt::Debug   应为所有公共类型实现，输出通常会尽可能真实地表示内部状态，Debug trait的目的 便于调试Rust代码，大多数情况下使用`#[derive(Debug)]`

### Related macros

相关的宏

> - format!
> - write!          第一个参数是目标的&mut io::Write
> - writeln!       在write!的基础上，添加一个换行符
> - print!            格式字符串将打印到标准输出
> - println!         在print!基础上添加一个换行符
> - eprint!            格式字符串打印为标准错误
> - eprintln!         相似于eprint! 在其基础上添加换行符
> - format_args!      用于安全地传递描述格式字符串的不透明对象。**此对象不需要任何堆分配来创建，它只引用堆栈上的信息**。

#### write! 

```rust
use std::io::Write;
let mut w = Vec::new();
write!(&mut w, "Hello {}!", "world");
```

#### format_args!

```rust
use std::fmt;
use std::io::{self, Write};

let mut some_writer = io::stdout();
write!(&mut some_writer, "{}", format_args!("print with a {}", "macro"));

fn my_fmt_fn(args: fmt::Arguments) {
    write!(&mut io::stdout(), "{}", args);
}
my_fmt_fn(format_args!(", or a {} too", "function"));
```