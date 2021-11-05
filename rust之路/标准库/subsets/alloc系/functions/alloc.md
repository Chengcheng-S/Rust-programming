### std::alloc::alloc

```rust
pub unsafe fn alloc(layout:Layout)->*mut u8
```

使用全局分配器分配内存

> 此函数会将调用转发到已通过`#[global_allocator]`属性注册的分配器的`GlobalAlloc::alloc`方法(若存在的话)， 或者是`std`crate的默认值。当该函数和`AllocRedtrait`稳定时,不建议使用此函数，而是去使用Global的`alloc`方法。

eg:

```rust
use std::alloc::{alloc,dealloc,Layout};

fn main() {
    unsafe {
        let la=Layout::new::<u16>();
        let ptr=alloc(la);
        *(ptr as *mut u16)=42;
        assert_eq!(*(ptr as *mut u16),42);
        dealloc(ptr,la);
    }
}
```

### safety

`GlobalAlloc::alloc`

