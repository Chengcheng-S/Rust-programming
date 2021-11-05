# crate

Rust的编译单元，当调用`rustc name.rs`时 `name.rs`被当作**crate文件**，如果`name.rs`含有`mod`声明，那么模块文件的内容建在编译之前被插入crate文件的相应声明处，**模块不会被单独编译，只有crate时才会被编译**。

crate 可以编译成二进制可执行文件（binary）或库文件（library）。默认情况 下，`rustc` 将从 crate 产生二进制可执行文件。这种行为可以通过 `rustc` 的选项 `--crate-type` 重载。

## 库

```
rustc --crate-type=lib name.rs
cd lib*
```

默认情况下，库会使用crate文件中的迷你工资，前面加上lib前缀，但这个默认名称可以使用`crate_name`属性覆盖。

## extern crate

创建库连接到一个crate 必须使用`extern crate`声明，这不仅会链接库，还会用一个与库名相同的模块来存放库里的所有项，于模块的可见性规则也适用于库。

```rust
// 链接到 `rary` 库，导入其中的项
extern crate rary;
fn main() {
    rary::public_function();
    // 报错！ `private_function` 是私有的
    //rary::private_function();
    rary::indirect_access();
}
```

```
rustc executable.rs --extern rary=library.rlib && ./executable
```





