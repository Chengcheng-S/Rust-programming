# alloc

内存分配API

在给定的程序中，标准库有一个“全局的内存分配器”，当前默认的全局分配器未指定。

## #[global_allocator]

此属性允许配置全局分配器的选择，可以使用它来实现一个完全自定义的全局分配器，将所有默认的分配请求路由到一个自定义对象。



```

```

## struct

### layout

内存块布局

### layoutErr

给`Layout::from_size_align`或者其他布局构造函数的参数不满足其文档化的约束。

### System

操作系统提供的默认内存分配器

### AllocErr

表示分配失败的错误，原因： 资源耗尽或者将给定的输入参数与此分配器组合时出现错误

### Global

全局内存分配器

### MemoryBlock

表示分配器返回的已分配内存块

## Enums

### Alloclnit

分配内存所需的初始状态

### ReallocPlacement

增加或缩小现有分配时的放置限制

## Traits

### GlobalAlloc

一个内存分配器，可以通过#[global_allocator]属性

### AllodRef

实现其可以分配、增长、收缩和释放通过布局描述的任意数据块

## functions

### alloc

使用全局分配器分配内存

### alloc_zeroed

使用全局分配器分配置零初始化内存

### dealloc

使用全局分配器释放内存

### handle_alloc_error

内存分配错误或失败时终止



### realloc

使用全局分配器重新分配内存

### set_alloc_error_hook

注册一个自定义分配错误钩子，替换以前注册的任何钩子

### take_alloc_error_hook

注销当前分配错误的钩子，并返回它

