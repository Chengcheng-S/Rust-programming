# Cargo

`cargo` 是官方的 Rust 包管理工具。

- 依赖管理和与cargo.io 集成
- 方便单元测试
- 基准测试

## 依赖

新建Rust 项目

```
cargo new name
```

新建库

```
cargo new --lib name 
```

执行之后生成

```
name
├── Cargo.toml
└── src
    └── main.rs
```

- main.rs ===> 项目的入口源文件
- cargo.toml=====> 配置文件

cargo.toml

```
[package]
name = "name"
version = "0.1.0"
authors = ["mark"]
[dependencies]
```

- `package`下的`name` 表示项目的名称
- version 使用语义版本控制的crate版本号
- `author` crate 作者
- `dependencies` 为项目添加依赖

构建项目： `cargo build`

cargo run 运行

cargo  check 检测程序

## 约定规范

默认二进制名称是`main`，但可以通过将文件放在`bin`目录中来添加其他二进制可执行文件：

```rust
foo
├── Cargo.toml
└── src
    ├── main.rs
    └── bin
        └── my_other_bin.rs
```

为了使得`cargo`**编译或运行这个二进制可执行文件**而不是默认或其他二进制可执行文件，只需给cargo增加一个参数`–bin my_other_bin`其中`my_other_bin`则是想要使用的二进制文件的名称。

## 测试

Rust对单元和集成测试提供了一流的支持。

在项目目录层上，可以将单元测试放在需要测试的模块中，并将集成测试放在源码中`tests/`目录中

```
foo
├── Cargo.toml
├── src
│   └── main.rs
└── tests
    ├── my_test.rs
    └── my_other_test.rs
```

tests目录下的每个文件都是一个单独的集成测试，`cargo`提供了一种便捷的方式来运行测试

```
cargo test
```

匹配某个一个模式

```
cargo test test_demo
```

注：`cargo`可能同时进行多项测试，因此需要确保其不会互相竞争(如：将其输出到文件中，应写入到不同的文件)

## 构建脚本

构建cargo可以运行的脚本。

需要像包中添加构建脚本，可以在`cargo.toml`中指定

```rust
[package]

build = "build.rs"
```

不同于默认情况，cargo将在项目目录中优先查找`build.rs`文件。

构建脚本指示另一个Rust文件，此文件将在编译包中的任何其他内容之前，**优先进行编译和运行**，因此，此文件可实现满足crate的先决条件。

cargo 通过此处指定的可以使用的**环境变量**为脚本提供**输入**，**此脚本通过stdout提供输出**。

此外， `cargo`为前缀的行将由cargo直接解析，因此用于定义包编译的参数。

## 属性

属性是应用于某些模块,crate或项的元数据(metadata)，这些数据可以用来：

- 条件编译代码
- 设置crate名称版本和类型(二进制文件或库)
- 禁用lint(警告)
- 启用编译器的特性(宏、全局导入(glob import)等)
- 连接到一个非Rust语言的库
- 标记函数为单元测试
- 标记函数作为基准测试的某个部分

当属性作用于**整个crate**时，他们的语法为`#![crate_attribute]`, 当其用于**模块或项**时，语法为`#[item_attribute]`（注意没有感叹号）

属性可以接收参数，有不同的语法形式：

- `#[attribute = "value"]`
- `#[attribute(key="value")]`
- `#[attribute(value)]`

属性可以有多个值，其可以分开到多行中

```rust
#[attribute(value,value2)]

#[attribute(value,value,
			value,value)]
```

### dead_code

编译器提供了`dead_code`死代码/无效代码，这会对未使用的函数产生警告，可以使用一个属性来禁用这个`lint`。

```rust
fn f1(){}

#[allow(dead_code)]
	
fn f2(){}

fn f3(){}

fn main(){f1();}
```

#[allow(dead_code)]属性可以禁用 `dead_code`lint

在实际过程中，需要将死代码清除掉。

### crate

`crate_type`属性可以告知编译器crate是一个二进制的可执行文件还是一个库(以及种类)，

`crate_name`属性可以设置crate的名称

注：在使用cargo时，这两种类型是都没有作用，由于大部分的Rust工程都是用cargo，这意味着crate_type 和crate_name 的作用事实上很有限。

```rust
#![crate_type="lib"]
//  这个crate是一个库文件

#![crate_name="rary"]
// 库的名称rary

pub fn pubilc_function(){
    println!("called rary's `public_function()`");
}
fn private_function(){
    println!("called rary's `private_function()`");
}

pub fn indirect_function(){
    println!("called rary's `indirect_function` that");
	private_function();
}
```

注： 当使用`crate_type`属性时，就不再需要给`rustc`命令加上`--crate-type`标记



### cfg

条件编译通过以下两种不同的操作符实现：

- cfg 属性： 在属性的位置中使用`#[cfg...]`
- cfg!宏： 在bool表达式中使用`cfg!(...)`

```rust
#[cfg(target_os="linux")]  / 该函数仅在liunx的时候才会编译
fn f3(){println!("this is rust for linux");} 
```

```rust
/ 而这个函数仅当目标系统 **不是** Linux 时才会编译
#[cfg(not(target_os = "linux"))]
fn are_you_on_linux() {
    println!("You are *not* running linux!")
}
```

```rust
fn main() {
    are_you_on_linux();
    println!("Are you sure?");
    if cfg!(target_os = "linux") {
        println!("Yes. It's definitely linux!");
    } else {
        println!("Yes. It's definitely *not* linux!");
    }
}
```

### 自定义条件

部分条件如`target_os`由`rustc`隐式提供,但自定义的条件必须使用`--cfg`标记来传给`rustc`。

```rust
#[cfg(some_condition)]
fn conditional_func(){
	println!("condition met!");
}

fn main(){
    conditional_function();
}
```

使用自定义的cfg标记

```rust
rustc --cfg some_condition custom.rs && ./custom
condition met!
```

未使用标记

```
rustc custom.rs && ./custom
No such file or directory (os error 2)
```
