# Literals

## C++

### Integers

整数是一个十进制，其后有可选类型的后缀。

在C++中，整数字面量可以用数字表示，也可以用后缀表示，十六、八、二进制的值用前缀表示

```rust
// Integers
42
999U
43424234UL
-3456676L
329478923874927ULL
-80968098606968LL
// C++14
329'478'923'874'927ULL
// Hex, octal, binary
0xfffe8899bcde3728 // or 0X
07583752256
0b111111110010000 // or 0B
```

整数的u、l和ll后缀表示其是无符号的、long还是long-long类型。u和ll可以是大写或小写。通常U必须在size之前，C++14 允许相反的顺序。

C++ 14还允许单引号插入到分隔符中，这些音号可以出现在任何地方，并且被忽略。

### Float point number

浮点数可以表示整数或小数。 

### Boolean values

C/C++布尔类型的字面量是`true`  `false`

### Characters and Strings

字符的字面量由单引号和可选宽度前缀括起来。前缀`L`表示宽字符，`u`表示UTF-16，`U`表示UTF-32。

```rust
'a'
L'a' // wchar_t
u'\u20AC' // char16_t
U'\U0001D11E' // char32_t
```

注：

- sieof(‘a’)在C中产生sieof(int)，而C++中则是sieiof(char)。
- char16和char32分别足以容纳任何UTF-16和UTF-32代码单元

字符串是用双引号括起来的字符序列。零值终止符总是附加到结尾。前缀的工作原理与具有附加u8类型的字符文本相同，以指示UTF-8编码的字符串。

```c++
"Hello"
u8"Hello" // char with UTF-8
L"Hello"   // wchar_t
u"Hello"   // char16_t with UTF-16
U"Hello"   // char32_t with UTF-32
```

### User-defined literals

C++ 11引入了用户定义的文字。它们允许整数、浮点数、字符和字符串具有由下划线和小写字符串组成的用户定义的类型后缀。前缀可以充当修饰符的形式，甚至可以充当在编译时修改值的常量表达式运算符。

C++ 14进一步定义了复杂的数字和时间单位的用户定义文字。

## Rust

### Integers

在Rust中，数字字面量也可以仅表示为数字，也可以用后缀表示。十六进制、八进制和二进制的值也用前缀表示：

```rust
// Integers
123i32;
123u32;
123_444_474u32;
0usize;
// Hex, octal, binary
0xff_u8;
0o70_i16;
0b111_111_11001_0000_i32;
```

Rust中下划线是一个分隔符，其功能与C++中的单引号相同。

### Floating point numbers

浮点数可以表示整数或小数，与整数一样，他们也可以加后缀来表示其类型

```rust
let a = 100.0f64;
let b = 0.134f64;
let c = 2.3f32; // But 2.f32 is not valid (note 1)
let d = 12E+99_E64;
```

### Booleans

布尔类型的字面量就是`true` `false`

### Characters and Strings

Rust中的字符是由的那引号引起来的任何UTF-32代码点。此值可以转义，也可以不转义，因为rs文件是UTF-8 编码的。

特殊前缀b可用于表示字节字符串，即每个字符为单个字节的字符串。



```rust
"This is a string"
b"This is a byte string"
```

前缀b表示字节字符串，即单字节字符。Read允许使用类似于C++的反斜杠符号来逃避新行、空间、双引号和反斜杠。

```rust
r##"This is a raw string that can contain \n, \ and other stuff without escaping"##
br##"A raw byte string with "stuff" like \n \, \u and other things"##
```