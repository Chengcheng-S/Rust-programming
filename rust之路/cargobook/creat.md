## creat

使用creat创建一个新的工程

```
cargo new name --bin
```

`--bin`参数 意味着编写二进制程序， 若要构建库需要传递`–lib` ,默认情况下回创建一个新的git仓库， 若不希望如此可以传入参数`--vcs none`

```
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

使用cargo build --release启用优化功能来编译文件

```
cargo build --release
```

将生成的二进制文件放入目标/发行版而不是目标/调试中。在调试模式下进行编译是默认的开发方式。由于编译器不进行优化，因此编译时间较短，但是代码运行速度较慢。发布模式需要更长的时间来编译，但是代码将运行得更快。

### Dependencies

crates.io 是Rust社区的包中心，可作为发现和下载软件包的位置。货物被配置为默认使用它来查找请求的包裹。可将项目依赖的库，将其添加到Cargo.toml

```rust
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

### 项目布局

Cargo使用约定进行文件放置，以使您轻松进入新的Cargo软件包：

```
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

- `cargo.toml` `cargo.lock` 存储项目的根目录
- 源代码放置在src目录中
- 默认的库文件是`src/lib.rs`
- 默认的可执行文件是`src/main.rs`
  - 其余的可执行文件可以放置于`src/bin`
- 基准测试在`benches`目录中
- 示例放置于`Examples`中
- 集成测试放置在tests目录中

如果二进制，示例，基准或集成测试包含多个源文件，则将main.rs文件以及其他模块放在src / bin，examples，基准或tests目录的子目录中。可执行文件的名称将是目录名称。