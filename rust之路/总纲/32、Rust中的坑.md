## impl trait

impl trait 指定实现特定trait的未命名但具体类型的新方法。可以将其放在：参数位置和返回位置。

```rust
trait T{}
fn foo(arg:impl T){}

fn foo()->impl T{}
```

### 参数位置

```rust
trait T{}
fn foo(arg:impl T){}

fn foo<T:T>(arg:T){}
```

### 返回值位置

```rust
trait T{}

impl T for i32{}
fn foo()->Box<dyn T>{
    Box::new(5)
}
```

此处将会有一些开销，堆分配 Box<T>

```rust
trait T{}
impl T for i32{}
fn foo()->impl T{
    3
}
```

### 闭包

在Rust中，闭包具有唯一的，不可写的类型。但是，它们确实实现了Fn特质。这意味着以前，从函数返回闭包的唯一方法是使用trait对象。

```rust
fn foo()->Box<dyn Fn(i32)->i32>{
	Box::new(|x|x+1)
}
```

直接返回闭包类型

```rust
fn foo()->impl Fn(i32)->i32{
    |x| x+1
}
```

## dyn triat

功能：

- Box<Trait>  ===> Box<dyn Trait>
- &Trait  &mut Trait ===>&dyn Trait  &mut dyn Trait

```rust
trait T{}
impl T for i32{}
//old
fn foo()->Box<T>

// new 
fn foo()->Box<dyn T>{}
```

## 容器对象

```rust
use std::rc::Rc;
impl Foo for i32{}

fn main(){
    let obj:Rc<dyn Foo> = Rc::new(5);
}
```

## 关联类型

对于 struct、trait、enums可以定义关联类型

```rust
struct A;
impl A{
    fn foo(){
        println!("associated constants");
    }
}
fn main(){
    A::foo();
}
```

## 匿名特征函数

特征方法声明中的参数不再允许匿名 

必须为所有参数指定参数名称

```rust
fn f1(a:i32){}
```

## Slice patterns

对切片内容和结构进行匹配

```rust
fn main() {
    let a=["hello","rust","lang"];
    fa_match(&a);
}
fn fa_match(a:&[&str]){
    match a{
        []=>println!("empty"),
        [one]=>println!("one element {}",one),
        [two,three]=>println!("two elements {},{}",two,three),
        [_,_,good]=>println!("good {}",good),
        _=>panic!("唉 出错了"),
    }
}
```

## 声明周期和所有权

### 非词汇生命周期

通过称为“非词法生存期”的机制，借用检查器得到了增强，可以接受更多代码。生命周期遵循“词汇范围”.

```rust
fn main(){
	let mut x =5;
	let y=&x;
	let z=&mut x
}
```

### 默认匹配绑定

> 在Rust 2018中，要匹配的值的类型会通知绑定模式，因此，如果与带有Some变体的＆Option <String>匹配，则会自动进入ref模式，从而为提供内部数据的借用视图。同样，＆mut Option <String>将为您提供ref mut视图

### 匿名生命周期

`‘_` 匿名生命周期

Rust 2018允许为可能不清楚的类型显式标记删除生存期的位置。为此，可以使用特殊的生存期'_，就像可以使用let x语法显式标记一个类型被推断出来一样：_ = ..;

```rust
impl<T> Drop for SetOnDrop<'_, T> {
    fn drop(&mut self) {
        if let Some(x) = self.value.take() {
            *self.borrow = x;
        }
    }
}
```



### 结构体中的生命周期

```rust
// Rust 2018

struct Ref<'a, T> {
    field: &'a T
}

struct WhereRef<'a, T> {
    data: &'a T
}

struct RefRef<'a, 'b, T> {
    field: &'a &'b T,
}

struct ItemRef<'a, T: Iterator> {
    field: &'a T::Item
}
```

### 静态和常量的生命周期更短

```rust
const NAMES: &[&str; 2] = &["Ferris", "Bors"];
```

## 数据类型

### 字段初始化

两个变量具有相同的名称，可以不写两个变量

```rust

#![allow(unused)]
fn main() {
struct Point {
    x: i32,
    y: i32,
}

let x = 5;
let y = 6;

// new
let p = Point {
    x,
    y,
};
}
```

## ..= 包含范围

```rust
for i in 1..3 {
    println!("i: {}", i);
}
```

输出 1，2

```rust
#![allow(unused)]
fn main() {
for i in 1..=3 {
    println!("i: {}", i);
}
}
```

输出1，2，3

### 128位整数

Rust现在具有128位整数！

```rust
let x:i128=0;
let y:u128=0;
```

它们的大小是u64的两倍，因此可以容纳更多的值。进一步来说，

- `u128`: `0` - `340,282,366,920,938,463,463,374,607,431,768,211,455`
- `i128`: `−170,141,183,460,469,231,731,687,303,715,884,105,728` - `170,141,183,460,469,231,731,687,303,715,884,105,727`

### 实现Opertor-equals

各种`等于运算符`的运算符（例如+ =和-=）可以通过各种特征来实现。

```rust
use std::ops::AddAssign;

#[derive(Debug)]
struct Count{
    value:i32,
}
impl AddAssign for Count{
    fn add_assign(&mut self,other:Count){
        self.value+=other.value;
    }
}

fn main(){
	let mut c1=Count{value:1};
    let c2=Count{value:5};
    c1 += c2;
    println!("{:?}",c1);
}
```

### union是不安全的enum

Rust支持`union`

```rust
union A{
	f1:u32,
	f2:f32,
}
```

> union有点像枚举，但它们是“未标记的”。枚举有一个“标签”，可以在运行时存储哪个变体是正确的变量。union没有这个标签。由于可以使用错误的变体来解释union中保存的数据，而Rust无法为检查此数据，因此，读取**union字段是不安全的**

```rust
  unsafe {
        let mut a =A{f1:1};
        println!("{}",a.f1);
}
union A{
    f1:i32,
    f2:u32,
}
```

union主要用于C的交互，CAPI可以(并取决于区域)经常公开并集，使得为这些库编写API包装程序变得更加容易，

此外 union 简化了依赖于值表示得空间高效或缓存高效结构的Rust实现。 使用齐指针的最低有效来区分大小写的机器子大小写union，仍有带改进。

**目前union只能包含可复制类型，而不能实现Drop**。

### 选择与repr属性对齐

维基定义：

> 当数据自然对齐时，现代计算机硬件中的CPU最有效的执行对内存的读写操作，这通常意味者数据地址是数据大小的倍数。
>
> 数据对齐是根据元素的自然对齐来对齐元素。为了确保自然对齐，可能需要在结构元素之间或结构的最后一个元素之后插入一些填充

#[repr]属性具有一个参数align，它设置结构体的对齐方式

```rust
#![allow(unused)]
fn main() {
struct Number(i32);

assert_eq!(std::mem::align_of::<Number>(), 4);
assert_eq!(std::mem::size_of::<Number>(), 4);

#[repr(align(16))]
struct Align16(i32);

assert_eq!(std::mem::align_of::<Align16>(), 16);
assert_eq!(std::mem::size_of::<Align16>(), 16);
}
```

### SIMD 加快计算速度

SIMD 代表单指令 ，多个数据。

## Micro

在Rust中，可以通过generate属性自动实现某些trait

```rust
#[derive(Debug)]
struct Pet{
    name:String,
}
```

### 样式宏

macro_rules!

在Rust2018可以通过use语句而不是旧的#[macro_use]属性从外部引入特定的宏

```rust
#[macro_export]
macro_rules! baz {
    () => ()
}
```

```rust
// Rust 2018

use bar::baz;

fn main() {
    baz!();
}
```

注：仍需要使用#[macro_use]来使用自己的crate中定义的宏，此功能仅适用于外部引入的宏。

### 过程宏

使用过程宏派生特征时，必须命名提供自定义派生的宏，这通常与特征名称相匹配，需要确保提供衍生信息的crate文档进行检查。

### 本地帮助宏

有时在模块中包含帮助程序宏很有帮助或必要。这可能会使支持两种版本的Rust变得更加复杂。

不适用？作为分隔符，避免和？混淆

```rust
macro_rules! foo {
    ($a:ident $(, $b:expr)?) => {
        println!("{}", $a);

        $(
            println!("{}", $b);
         )?
    }
}
```

## 编译信息

### 改进错误信息

### 增量编译

### 弃用属性

弃用某些属性，使用`deprecated`

```rust
#[deprecated(
	since="0.2.1",
    note="Please use the bar function instead"
)]
pub fn foo(){}
```

```
   Compiling playground v0.0.1 (file:///playground)
warning: use of deprecated item 'foo': Please use the bar function instead
  --> src/main.rs:10:5
   |
10 |     foo();
   |     ^^^
   |
   = note: #[warn(deprecated)] on by default
```

## Rustup

Rust 版本管理器，

通过rustup 安装Rust

```
rustup toolchain install 1.4.8
```

最近的版本

```
rustup toolchain install nightly-2020-09-29
```

更新：

```
rustup update
```

安装组件

```
rustup components list
```

rust doc 本地文档

rust-src Rust 源码副本

```
rustup component add rust-src
```

rust-fmt 自动格式化程序

```
cargo fmt
```

继承IDE rls

```
rustup component add rls
```

检查程序

```
rustup component add clippy
```

## cargo

### cargo check 检查程序

### cargo install  安装

Cargo增加了新的安装命令。它旨在用于为Cargo安装新的子命令，或为Rust开发人员安装工具。

### 扩展

作为扩展Cargo的示例，您可以使用cargo-update软件包

```
cargo install cargo-update
```

使用cargo install-update -a命令，该命令会检查已进行cargo安装的所有内容并将其更新为最新版本。

创建默认的库文件

### cargo new

cargo new将默认生成二进制文件，而不是库文件。 

cargo new接受两个标志：--lib（用于创建库）和--bin（用于创建二进制文件或可执行文件）

### cargo rustc

将所有的标志传递给rustc

cargo rustc是Cargo的新子命令，它允许通过Cargo传递任意rustc标志，使用print-type-sizes来查看类型具有哪些布局信息，则可以运行此命令。

```
cargo rustc -- -Z print-type-size
```

注：cargo rustc仅将这些标志传递给crate调用，而不传递给用于构建依赖项的任何rustc调用。

### 多包装项目的cargo workspaces

```
my-package
 └──src
     └── lib.rs // code here
 └──examples 
     └── simple-example.rs // a single-file example
     └── complex-example
        └── helper.rs
        └── main.rs // a more complex example that also uses `helper` as a submodule
```

### 用patch 替换dependencies

覆盖依赖关系图的某些部分时，可以使用Cargo.toml的[patch]部分。

```
[dependencies]
foo = "1.2.3"
```



```
[dependencies]
foo = "1.2.3"

[patch.crates-io]
bar = { path = '/path/to/bar' }
```

### 使用本地注册表

```rust
[source.crates-io]
replace-with = 'my-awesome-registry'

[source.my-awesome-registry]
registry = 'https://github.com/my-awesome/registry-index'
```

### Crates.io禁止使用通配符

Crates.io不允许上传具有通配符依赖项的软件包。换句话说，这些：

```
[dependencies]
regex = "*"
```

代替 >，<=，以及所有其他非*范围也适用

```rust
[dependencies]
regex = "1.0.0"
```

## 文献

```
rustup doc 
```

### 库文件分层

标准库分为两层：有一个小型核心库`libcore`，在其基础上构建了完整的标准库libstd，

libcore完全和平台无关，并且仅需要定义一些外部符号，Rust的libstd建立在libcore之上，增加了对内存分配和I/O之类的支持。

在嵌入式空间之中使用Rust的应用程序以及编写操作系统的应用程序仅使用libcore 避开 libstd

，尽管现在支持使用libcore构建库，但是构建完整的应用程序还不稳定。

要使用libcore，请将此标志添加到crate根目录：

```
#![no_std]
```

将删除标准库，并将核心crate引入命名空间以供使用

```rust
#![no_std]
use core::cell::Cell
```

查看核心标准库 [查](https://doc.rust-lang.org/core/)

### WebAssembly 支持

编写Rust代码，并使它生成asm.js（wasm的先驱）和/或WebAssembly

```
$ rustup target add wasm32-unknown-emscripten
$ echo 'fn main() { println!("Hello, Emscripten!"); }' > hello.rs
$ rustc --target=wasm32-unknown-emscripten hello.rs
$ node hello.js
```

## 全局分配器

分配器是Rust中的程序在运行时从系统获取内存的方式。

＃[global_allocator]属性，允许Rust程序将其分配器设置为系统分配器，并通过实现GlobalAlloc特性来定义新的分配器。

通过声明静态变量并使用＃[global_allocator]属性对其进行标记，从而在需要时用于切换至系统分配器。

```rust
use std::alloc::System;

#[global_alloctor]
static GLOBAL:System=System;
fn main(){
    let mut v=Vec::new();
    v.push(1);
}
```

## 与C交互

cdylib create 实现与C的交互

如果要生成打算从C（或通过C FFI使用另一种语言）使用的库，则Rust无需在最终目标代码中包含特定于Rust的内容。对于类似的库，需要在Cargo.toml中使用cdylib crate类型：

```rust
[lib]
crate-type =["cdylib"]
```

这将产生一个较小的二进制文件，其中没有Rust特定的信息。

## dbg!

dbg！宏提供了比println！更好的调试体验：

```rust
fn main(){
	let x=5;
    dbg!(x);
}
```

```
[src/main.rs:4] x = 5
```

将获得调用它的文件和行号，以及名称和值。另外，println！打印到标准输出，因此您确实应该使用eprintln！打印到标准错误。 dbg！做正确的事，去stderr。

```rust
#![allow(unused)]
fn main() {
fn factorial(n: u32) -> u32 {
    if dbg!(n <= 1) {
        dbg!(1)
    } else {
        dbg!(n * factorial(n - 1))
    }
}
}
```

```
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = true
[src/main.rs:4] 1 = 1
[src/main.rs:5] n * factorial(n - 1) = 2
[src/main.rs:5] n * factorial(n - 1) = 6
[src/main.rs:5] n * factorial(n - 1) = 24
[src/main.rs:11] factorial(4) = 24
```

由于dbg! 将返回其调试值，而不是eprintln! 如果返回的是（），则无需更改代码的结构。

### 程序中使用jemalloc 分配器

```
jemallocator = "0.1.8"
```

或者添加到程序中

```rust
#[global_allocator]
static ALLOC: jemallocator::Jemalloc = jemallocator::Jemalloc;
```

## 统一路径

允许以前无效的导入路径语句的解析方式与非导入路径完全相同。

```rust
#![allow(unused)]
fn main() {
enum Color {
    Red,
    Green,
    Blue,
}

use Color::*;
}
```

## 文字宏匹配

为宏添加了一个新的文字匹配器

```rust
macro_rules! m {
    ($lt:literal) => {};
}

fn main() {
    m!("some string literal");
}
```

文字匹配任何类型的文字；字符串文字，数字文字，char文字。

## 宏中的？

macro_rules可以使用`?`

```rust
macro_rules! bar{
	($(a)?)=>{}
}
```

？将匹配模式的零个或一个重复，类似于已经存在的*表示“零个或多个”，而+表示“一个或多个”。（类正则）

## const fn

const fn允许您在“ const context”中执行代码。

### 整数运算符

可以对整数文字进行算术运算

```rust
const fn foo()->u32{
	5+6
}
```

### 布尔运算符

```rust
const fn mask(val: u8) -> u8 {
    let mask = 0x0f;

    mask & val
}

```

可以使用&&和||以外的布尔运算符，因为它们会短路评估：

```rust
macro_rules! m {
    ($lt:literal) => {};
}
fn main() {
    let a = ["hello", "rust", "lang"];
    fa_match(&a);
}

fn fa_match(a: &[&str]) {
    match a {
        [] => println!("empty"),
        [one] => println!("one element {}", one),
        [two, three] => println!("two elements {},{}", two, three),
        [_, _, good] => println!("good {}", good),
        _ => panic!("唉 出错了"),
    }
    unsafe {
        let a = A { f1: 1 };
        dbg!("{}",a.f1);
    }
    foo(fa());
    m!("some value in rust programming");
}

union A {
    f1: i32,
    f2: u32,
}

fn foo(n: u32) -> u32 {
    if dbg!(n < 1) {
        dbg!(1)
    } else {
        dbg!(n * foo(n - 1))
    }
}

const fn fa() -> u32 { 5 + 6 }
```

### 构造数组，结构，枚举和元组

```rust
struct Point {
    x: i32,
    y: i32,
}

enum Error {
    Incorrect,
    FileNotFound,
}

const fn foo() {
    let array = [1, 2, 3];

    let point = Point {
        x: 5,
        y: 10,
    };

    let error = Error::FileNotFound;

    let tuple = (1, 2, 3);
}

```

### 调用其他的const fn

```rust
const fn foo() -> i32 {
    5
}

const fn bar() -> i32 {
    foo()
}

```

### 数组 切片上的索引表达式

```rust
const fn foo() -> i32 {
    let array = [1, 2, 3];

    array[1]
}
```

### 结构和元组的现场访问

```rust
struct Point {
    x: i32,
    y: i32,
}

const fn foo() {
    let point = Point {
        x: 5,
        y: 10,
    };

    let tuple = (1, 2, 3);

    point.x;
    tuple.0;
}
```

### 从常量读取

```rust
const FOO: i32 = 5;

const fn foo() -> i32 {
    FOO
}

```

### & 和* 引用

```rust
const fn foo(r: &i32) {
    *r;

    &5;
}
```

### 强制转换

但指向整数强制转换的原始指针除外

```rust
const fn foo() {
    let x: usize = 5;

    x as i32;
}
```

### 不可辩驳的破坏模式

```rust
const fn foo((x, y): (u8, u8)) {
    // ...
}

```



### let 绑定

```rust
const fn foo() {
    let x = 5;
    let mut y = 10;
}
```

### 分配

```rust
const fn foo() {
    let mut x = 5;
    x = 10;
}
```

### 调用不安全的 fn

```rust
const unsafe fn foo() -> i32 { 5 }

const fn bar() -> i32 {
    unsafe { foo() }
}
```



```rust
#![allow(unused)]
fn main() {
trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<dyn FnBox + Send + 'static>;
}
```

































