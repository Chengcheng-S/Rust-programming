# Cargo

cargo是rust工具链中内置的构建系统以及包管理器。

终端输入

```
cargo --version 
```

检查cargo是否已经安装，往后的开发cargo是一个必不可少的项目管理器。

## 使用cargo创建项目

终端输入

```
cargo new project_name
```

进入到创建的目录文件夹会发现，生成了`cargo.toml` ，以及一个src的目录，其中包含`main.rs`文件,并且cargo还会初始化一个新的git仓库，默认生成`.gitgnore`文件。

注：git是一种常见的版本管理系统，在创建项目时可以使用`--vcs`选择不使用git,或者指定版本的git。运行命令cargo new --help 可以获取命令参数更多的说明

### cargo.toml 配置详解

打开cargo.toml

```
[package]
name = "hello_crgo"
version = "0.1.0"
authors = ["name"]
edition = "2018"


[dependencies]

```

#### [package] 段落

`[package]`是一个区域标签,表明接下来的语句会被用于配置当前的程序的包，随后便是cargo编译程序时的配置信息：程序名、版本号、作者信息，程序使用的编译器版本。

```
[package]
 # 软件包名称，如果需要在别的地方引用此软件包，请用此名称。
name = "hello_world"
# 当前版本号，这里遵循semver标准，也就是语义化版本控制标准。
version = "0.1.0"    # the current version, obeying semver
# 软件所有作者列表
authors = ["you@example.com"]
# 非常有用的一个字段，如果要自定义自己的构建工作流，
# 尤其是要调用外部工具来构建其他本地语言（C、C++、D等）开发的软件包时。
# 这时，自定义的构建流程可以使用rust语言，写在"build.rs"文件中。
build = "build.rs"
# 显式声明软件包文件夹内哪些文件被排除在项目的构建流程之外，
# 哪些文件包含在项目的构建流程中
exclude = ["build/**/*.o", "doc/**/*.html"]
include = ["src/**/*", "Cargo.toml"]
# 当软件包在向公共仓库发布时出现错误时，使能此字段可以阻止此错误。
publish = false
# 关于软件包的一个简短介绍。
description = "..."
# 下面这些字段标明了软件包仓库的更多信息
documentation = "..."
homepage = "..."
repository = "..."
# 顾名思义，此字段指向的文件就是传说中的ReadMe，
# 并且，此文件的内容最终会保存在注册表数据库中。
readme = "..."
# 用于分类和检索的关键词。
keywords = ["...", "..."]
# 软件包的许可证，必须是cargo仓库已列出的已知的标准许可证。
license = "..."
# 软件包的非标许可证书对应的文件路径。
license-file = "..."
```



#### 依赖配置

`[dependencies]`,也是一个区域标签表明随后的区域被用来声明项目的依赖，存放第三方包之类的信息。

与平台的依赖定义格式不变，不同的是需要定义在[target]字段下：

```
# 注意，此处的cfg可以使用not、any、all等操作符任意组合键值对。
# 并且此用法仅支持cargo 0.9.0（rust 1.8.0）以上版本。
# 如果是windows平台，则需要此依赖。
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"
[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"
#如果是32位平台，则需要此依赖。
[target.'cfg(target_pointer_width = "32")'.dependencies]
native = { path = "native/i686" }
[target.'cfg(target_pointer_width = "64")'.dependencies]
native = { path = "native/i686" }
# 另一种写法就是列出平台的全称描述
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"
[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
# 如果使用自定义平台，请将自定义平台文件的完整路径用双引号包含
[target."x86_64/windows.json".dependencies]
winhttp = "0.4.0"
[target."i686/linux.json".dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
openssl = "1.0.1"
native = { path = "native/x86_64" }
# [dev-dependencies]段落的格式等同于[dependencies]段落，
# 不同之处在于，[dependencies]段落声明的依赖用于构建软件包，
# 而[dev-dependencies]段落声明的依赖仅用于构建测试和性能评估。
# 此外，[dev-dependencies]段落声明的依赖不会传递给其他依赖本软件包的项目
[dev-dependencies]
iron = "0.2"
```

### 自定义编译器调用方式模板详细参数

cargo内置**五**种编译器调用模板:dev 、release、bench、test、doc。

分别用于定义不同类型生成目标时的编译器参数，若要改变模板，可以自定义相应的字段即可。

- 开发模板，对应的cargo build 命令

  ```rust
  [profile.dev]
  opt-level =0   # 控制编译器的 --opt-level 参数 即 优化参数
  debug = true  #控制编译器是否开启-g 参数
  rpath =false  #控制编译器的-C rpath参数
  lto= false  # 控制-C lto 参数，影响可执行文件和静态库的生成
  debug-asserions=true #控制调试断言是否开启
  codegen-units =1 #控制编译器的 `-C codegen-units` 参数。注意，当`lto = true`时，此字段值被忽略
  ```

- 发布模板，对应的cargo build —release

  ```rust
  [profile.release]
  opt-level =3
  debug = false
  rpath = false
  lto = false
  debug-assertions = false
  codegen-units = 1
  ```

- 测试模板，对应的 cargo test 命令

  ```rust
  [profile.test]
  opt-level = 0
  debug = true
  rpath = false
  lto = false
  debug-assertions = true
  codegen-units = 1
  ```

- 性能评估模板  对应 cargo bench命令

  ```rust
  [profile.bench]
  opt-level = 3
  debug = false
  rpath = false
  lto = false
  debug-assertions = false
  codegen-units = 1
  ```

- 文档模板，对应的 cargo doc 命令

  ```rust
  [profile.doc]
  opt-level = 0
  debug = true
  rpath = false
  lto = false
  debug-assertions = true
  codegen-units = 1
  ```

- 注： 当调用编译器时，只有位于调用最顶层的软件包的模板文件有效，其他的子软件包或者依赖软件包的模板定义将被顶层软件包的模板覆盖

终端输入

```
cargo build
```

运行之后，在项目下创建一个名为`Cargo.lock`的文件，该文件记录了当前项目所有的依赖库以及具体版本号。

```
cargo run
```

则会运行项目，cargo还提供了一个代码检查的功能，在终端输入`cargo check`即可。

#### 以Release模式进行构建

在构建时可以使用

```
cargo build --release
```

在优化模式下构建并生成可执行文件，其生成的可执行文件会被放置在`target/release`文件下。

#### [features]段落

该段落中的字段被用于**条件编译**选项或者**可选依赖**

```rust
[package]
name ="noname"

[features]
#此字段设置了可选依赖的默认选择列表
# 注意这里的"session"并非一个软件包名称，
# 而是另一个featrue字段session
default = ["jquery", "uglifier", "session"]

```

```rust
#类似的值为空的feature 一般用于条件编译
#类似于`#[cfg(features="go-faster")]`
go-faster= []
```

```rust
# 此feature依赖于bcrypt软件包，
# 这样封装的好处是未来可以对secure-password此feature增加可选项目。
secure-password = ["bcrypt"]
```

```rust
# 此处的session字段导入了cookie软件包中的feature段落中的session字段
session = ["cookie/session"]
```



```rust
[dependencies]
# 必要的依赖
cookie = "1.2.0"
oauth = "1.1.0"
route-recognizer = "=2.1.0"
# 可选依赖
jquery = { version = "1.0.2", optional = true }
uglifier = { version = "1.5.3", optional = true }
bcrypt = { version = "*", optional = true }
civet = { version = "*", optional = true }
```

使用feature 遵循的规则：

- feature的名称在本描述中不能与出现的软件包名称冲突
- 除了default feature 其他所有的features均为可选的 
- features 不能相互循环包含
- 开发依赖包不能包含在内
- features组智能依赖于可选软件包

features的一个重要用途就是，当开发者需要对软件包进行最终的发布时，在进行构建时可以声明暴露给终端用户的features，这可以通过下述命令实现：

```
cargo build --release --features "shumway pdf"
```

#### 关于测试

运行cargo test 命令时，cargo将会按做以下事情：

- 编译并运行软件包源代码中被`#[cfg(test)]`所标志的单元测试
- 编译并运行文档测试
- 编译并运行集成测试
- 编译examples

### 配置构建目标

诸如[bin],[lib],[bench],[test]以及[example]等字段，均提供了类似的配置，以说明构建目标应如何被购标

```rust
[lib]
#库名称，默认与项目名称相同
name ="foo"


crate-type=["dylib"]
# 此选项仅用于[lib] 其决定构建目标的构建方式
# 可以取dylib (动态库)  rlib(r库)， staticlib (静态库)

path = "src/lib.rs"
#path 声明此构建目标对于cargo.toml文件的相对路径


test=true
# 单元测试的开关

doctest=true
# 文档测试的开关

bench=true
# 性能测试的开关


doc=true
# 文档生成开关选项

plugin=false 
# 是否构建为编译器插件的开关选项


harness=true
# 如果设置为false  cargo test 将会忽略传递给rustc 的 --test参数

```











