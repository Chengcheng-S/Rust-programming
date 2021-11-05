# Hello,World

默认读者已经搭建好Rust开发环境，编译器可以选择VIM，vscoed以及Jebtrains系列，不在赘述。

编程起源`Hello World`，创建一个"project"的文件夹，在该文件夹下，新建一个名为'main.rs'的文件，打开之后并且输入以下程序

```rust
fn main(){
	println!("hello,world!");
}
```

回到终端并且在该目录下执行

```
rustc main.rs
linux/mac的话执行
./main

windows 执行
main.exe
结果为
hello,world!
```

一切顺利的话，你完成了第一个Rust的程序，且正式成为了Rust开发者

对本程序的解析

```rust
fn main(){
	println!("hello,world");
}
```

没有参数、返回值的`mian`函数是程序的入口，`println!(...)`则是函数体，Rust要求他们应该位于`{}`之间，当然别忘了语句最后的`;`（表明语句的结束）。



