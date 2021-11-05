## Macros

### C++/C Preprocessor

C语言分两个阶段编译，第一阶段是`预处理` 在此阶段预处理器将查找以`#` 符号开头的指令，并基于这些指令运行字符串替换和条件包含/排除。只有文件经过预处理之后，编译器才会尝试编译。

预处理指令以`#`开头：

```C++
#define IS_WINDOWS
```

`#ifdef` `#else` `#endif` 根据匹配项包含测试的一个分支或另一个分支的代码。

```c++
#if SHAREWARE_VERSION == 1
showNagwarePopup();
#endif
//...
#ifdef IS_WINDOWS
writePrefsToRegistry();
#else
writePrefsTocfg();
#endif
```

再有就是`#include` 在C++/C中，公共函数和结构通常定义并在单独的文件中实现，`#include` 指令允许将头拉入任何使用这些定义文件的前面。

```c++
#include <string>
#include <stdio.h>

#include "MyClass.h"
```

### C/C++ Macros

宏是预处理器在编译源代码之前执行的字符串替换模式(容易出错，因此被弃用，转而使用常量和内联函数)

```c++
#define PRINT(x) \
  printf("You printed %d", x);
```



```c++
#define MULTIPLY(x,y) x*y

int x=10,y=20;
int result =MULTIPLY(x,y); 
```

- C++中的宏实现是不能传入表达式，编译器是没有如此智能的去处理表达式。
- 宏可能会去外部捕获值，从而导致代码中的错误

```c++
#define SWAP(x,y) int tmp = y ;y=x;x=y;

int tmp =10;
int a=20,b=30;
SWAP(a,b); //ERROR
```

```c++
#define SWAP(x, y) do { int tmp = y; y = x; x = y } while(0);
```

使用场景：

- 有条件地包含命令行标志或指令，例如编译器可以`#define WIN32`，这样代码就可以根据它的存在以某种方式有条件地编译。
- 根据`#define`（如是否设置调试）编译掉的内容

### Rust Macros

Rust中的宏比C++中的更安全

- 宏的卫生性：如果宏包含变量，它们的名称不会与使用它们的范围中的命名变量冲突、隐藏或以其他方式干扰。
- 在宏的方括号之间提供的模式被标记化并指定为Rust语言的一部分。
- 过程性：Rust宏要么是声明性的，要么是基于规则的，每个规则都有一个左侧的模式“matcher”和一个右侧的“substitution”。
- 宏必须产生语法正确的代码
- 声明性宏可以通过crates导出，并在其他代码中使用，前提是其他代码选择从crate启用宏支持。因为它必须用`#[macro_export]`进行声明。

```rust
marco_rules! hello{
    ($($name:expr),*)=>(
    	$(println!("Hello {}",name);)*
    )
}
hello!("jhon","su");
```

基本上匹配器与逗号分隔列表匹配，替换生成一个`println!()`

### 过程宏

因此，过程宏可以被认为是一个代码生成器，但它构成了实际编译器的一部分。程序宏特别适用于：

- 序列化/反序列化
- 特定语言 如：正则表达式
- 面向过程编程
- 新的lint规则
