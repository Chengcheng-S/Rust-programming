# std::thread_local

```rust
macro_rules! thread_local {
    () => { ... };
    ($(#[$attr:meta])* $vis:vis static $name:ident: $t:ty = $init:expr; $($rest:tt)*) => { ... };
    ($(#[$attr:meta])* $vis:vis static $name:ident: $t:ty = $init:expr) => { ... };
}
```

声明std:：thread:：LocalKey类型的新线程本地存储密钥。

### syntax

宏包装了任意数量的静态声明，并使它们成为线程本地的。允许每个静态的公开性和属性。

```rust
use std::cell::RefCell;
thread_local! {
    pub static FOO: RefCell<u32> = RefCell::new(1);

    #[allow(unused)]
    static BAR: RefCell<f32> = RefCell::new(1.0);
}
```





