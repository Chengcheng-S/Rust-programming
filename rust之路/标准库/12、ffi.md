# std::ffi

与FFI绑定相关的实用程序

该模块提供了处理非Rust接口上的数据的实用程序，与其他编程语言和底层操作系统一样。它主要用于需要与其他语言交换C类字符串的FFI（外部函数接口）绑定和代码。

Rust用String类型表示**拥有的字符串**，用str原语表示**借用的字符串**片段。

> 两者都是UTF-8编码，中间可能包含`nul`字节，也就是说，如果查看组成字符串的字节，其中可能有一个`\0`。String和str都显式地存储它们的长度；在字符串的末尾没有`nul`终止符，就像在C中一样。jj

C strings 和Rust strings 的不同：

- Encodings:  Rust strings是`utf-8`,但是C strings或许使用其他的编码格式。

- Character size    Cstrings 可以使用`char`或`wchar_t`大小的字符，(注：C的char 不同于Rust)   

  > - C标准允许解释这些类型的实际大小，但是为由每个字符类型组成的字符串定义了不同的API。Rust字符串总是UTF-8，因此不同的Unicode字符将以可变的字节数编码。Rust类型字符表示“Unicode标量值”，它与“Unicode码位”类似，但不相同。

- **Nul terminators and implicit string lengths**  (Nul终止符和隐式字符串长度)  

  > - C字符串以nul结尾，即结尾有一个\0字符。字符串缓冲区的长度不是存储的，而是必须计算的；要计算字符串的长度，C代码必须手动调用一个函数，如对基于字符的字符串调用strlen（），对基于wchar_t的字符串调用wcslen（）。这些函数返回字符串中不包括nul结束符的字符数，因此缓冲区长度实际上是len+1个字符。Rust字符串没有nul终止符；它们的长度总是存储的，不需要计算。在Rust中，访问字符串的长度是O（1）操作（因为长度是存储的）；在C中是O（长度）操作，因为长度需要通过扫描字符串的nul终止符来计算。

- Internal nul characters (内部nul字符串)，当C字符串有nul结束符字符时，这通常意味着它们中间不能有nul字符-nul字符实际上会截断字符串。Rust字符串的中间可以有nul字符，因为nul不必在Rust中标记字符串的结尾。

当需要使用C ABI 如Python在语言之间传输UTF-8字符串，需要使用`CString` 和`CStr`

- from Rust to C  ：CString表示一个拥有的、C友好的字符串：它以nul结尾，并且没有内部nul字符。Rust代码可以从普通字符串中创建一个CString(前提是字符串中间没有nul字符)，然后使用各种方法获取原始*mut u8，然后将其作为参数传递给对字符串使用C约定的函数。
- from C to Rust ：CStr代表一个借来的C字符串；它是用来包装从C函数获得的raw*const u8。CStr保证是以nul结尾的字节数组。一个CStr，如果它是有效的UTF-8，就可以将它转换为Rust&str，或者通过添加替换字符来无损地转换它。

OsStr   OsString 

当需要在操作系统本身之间传输字符串时，或者在捕获外部命令的输出时，OsString和OsStr非常有用。OsString、OsStr和Rust字符串之间的转换工作方式与CString和CStr的转换类似。

- OsString 以操作系统的任何表示形式表示拥有的字符串，在Rust标准库中，在操作系统之间传输字符串的各种api使用OsString而不是纯字符串。
- OsStr   表示对可以传递给操作系统的格式的字符串的借用引用。它可以转换成一个UTF-8 Rust String slice 类似于OsString。