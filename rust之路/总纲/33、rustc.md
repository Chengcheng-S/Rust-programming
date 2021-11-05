# Rustc

```
cargo build --verbose
```

查看项目中的依赖

### 用法

hello.rs

```rust
fn main(){
	println!("hello Rust");
}
```

将hello.rs编译为可执行文件

```
rustc hello.rs
```

便生成一个可执行的文件

注：只传递过crate根，而不是编译的每个文件。

如： 在foo.rs中定义

```rust
mod foo{
pub fn hello() {
    println!("Hello, world!");
	}
}
```

main.rs:

```rust
mod foo;

fn main() {
    foo::foo::hello();
}
```

只需要编译 main.rs即可

## Lint-levels

lint的级别

- allow
- warn
- deny
- forbid

### allow

默认情况下不执行任何操作

### warn

“warn”lint级别将生成警告。

```rust
#![allow(unused)]
fn main() {
pub fn foo() {
    let x = 5;
}
}
```

```rust
$ rustc lib.rs --crate-type=lib
warning: unused variable: `x`
 --> lib.rs:2:9
  |
2 |     let x = 5;
  |         ^
  |
  = note: `#[warn(unused_variables)]` on by default
  = note: to avoid this warning, consider using `_x` instead
```

### deny

若违反了deny lint 将会产生错误

```rust
fn main() {
    100u8 << 10;
}
```

```
$ rustc main.rs
error: bitshift exceeds the type's number of bits
 --> main.rs:2:13
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^
  |
  = note: `#[deny(exceeding_bitshifts)]` on by default
```

lint的错误与常规错误的区别： lint可以通过级别进行设置，因此与allow类似。

### forbid

它与“deny”相同，因为此级别的lint将生成错误，但与“deny”级别不同，“forbit”级别不能被重写为任何低于错误的级别。但是，lint级别可能仍有上限（见下文），因此将使lint设置为“forbid”仅警告。

```
--cap lintsrustc--cap lints warn
```

## 配置warning级别























