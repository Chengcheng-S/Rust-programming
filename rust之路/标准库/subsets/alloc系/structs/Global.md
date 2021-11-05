### std::alloc::Global

先行版API

全局内存分配

此类型通过将调用转发到使用`#[global_allocator]`属性注册的分配器，或`std crate`的默认值，来实现`AllocRef` tarit

注： 虽然这种类型不稳定，但是可以通过alloc中的自由函数所访问。

