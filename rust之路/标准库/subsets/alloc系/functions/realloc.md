### std::alloc::realloc

```rust
pub unsafe fn realloc(ptr:*mut u8,layout:Layout,new_size:usize)->*mut u8
```

使用全局分配器重分配内存。

> 此函数会将调用转发到已通过`#[global_allocator]`属性注册的分配器的`GlobalAlloc::alloc`方法(若存在的话)， 或者是`std`crate的默认值。当该函数和`AllocRedtrait`稳定时,不建议使用此函数，而是去使用Global的`alloc`方法

### safety

`GolbalAlloc::realloc`