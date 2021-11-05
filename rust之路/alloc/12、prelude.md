# alloc::prelude

此模块的目的是通过在模块顶部添加glob导入来减轻alloc crate常用方法的导入

```rust
#![feature(alloc_prelude)]
extern crate alloc;
use alloc::prelude::v1::*;
```

### Modules

v1                       第一个版本的prelude的alloc crates。