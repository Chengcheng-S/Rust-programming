# std::prelude

prelude是Rust自动导入到每个Rust程序中的东西的列表。尽可能地小，并且专注于一些东西，尤其时triat，这些特性在几乎每一个防锈程序中都会用到。这些特性在几乎每一个Rust程序中都会用到。

### Other Prelude

prelude看作一种模式，使得使用多种类型更加便捷(std::io::prelude io的prelude)

“prelude”与其他prelude的区别在于它们不是自动使用的，必须手动导入。这仍然比进口它们的所有组成部分容易。

### Prelude Contents

prelude的当前版本（版本1）位于std:：prelude:：v1中，并重新导出以下内容。

- `std::marker::{Copy,Send,Sized,Sync,Unpin}` 标记性状表明了类型的基本特性。
- `std::ops::{Drop,Fn,FnMut,FnOnce}`析构函数和重载（）的各种操作。
- `std::mem::drop` 显式删除值得函数
- `std::boxed::Box`  堆上分配值得方法
- `std::borrow::ToOwned` 定义为`to_owned`的转换trait，用于从借用的类型创建拥有的类型的泛型方法。
- `std::clone::Clone`定义克隆的普遍trait，即产生值副本的方法。
- `std::cmp::{ParticalEq,PartialOrd,Eq,Ord}`比较特征，它实现比较运算符，通常出现在特征边界中
- `std::convert::{AsRef,AsMut,Into,From}`泛型方法的作者用来创建泛型API转换
- `std::default::Default`  具有默认值的类型
- `std::iter::{Iterator,Extend,intointerator,doubleendeditor,ExactSizeIterator`各种迭代器
- `std::option::Option::{self, Some, None}.` 一种表示值的存在或不存在的类型。这种类型非常常用，它的变体也会导出。
- `std::string::{String, ToString}` 对分配得字符类型
- `std::vec::Vec` 一个可增长的堆分配向量。

### Modules

v1     Rust标准库prelude的第一个版本。