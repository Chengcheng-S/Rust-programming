# alloc::alloc

内存分配API

### structs

| Layout    | 内存块布局                                                   |
| --------- | ------------------------------------------------------------ |
| LayoutErr | `Layout:：from_size_align`或其他布局构造函数的参数不满足其文档化的约束 |
| AllocErr  | alloccerr错误表示分配失败，这可能是由于资源耗尽或将给定的输入参数与此分配器组合时出现错误所致。 |
| Global    | 全局内存分配器。                                             |

### Traits

| GlobalAlloc | 内存分配器，通过`#[global_allocator]`属性设置                |
| ----------- | ------------------------------------------------------------ |
| AllocRef    | `实验：`AllocRef的实现可以分配、增长、收缩和释放通过布局描述的任意数据块 |

### Functions

| alloc  （unsafe）        | 使用全局分配器分配内存         |
| ------------------------ | ------------------------------ |
| alloc_zeroed  （unsafe） | 使用全局分配器分配零初始化内存 |
| dealloc   （unsafe）     | 使用全局分配器释放内存         |
| handle_alloc_error       | 内存分配错误或失败时中止       |
| realloc  （unsafe）      | 使用全局分配器重新分配内存。   |

