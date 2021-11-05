## std::alloc::GlobalAlloc

```rust
pub unsafe trait GlobalAlloc{
    unsafe fn alloc(&self,layout:Layout)->*mut u8;
    unsafe fn dealloc(&self ,ptr:*mut u8,layout:Layout);
	unsafe fn alloc_zeroed(&self,layout:Layout)->*mut u8;
    
    unsafe fn realloc(
    	&self,
        ptr:*mut u8,
        layout:Layout,
        new_size:usize
    )->*mut u8{}
}
 
```

通过`#[global_allocator]`属性将内存分配器注册为标准库默认。

有些方法要求当前通过分配器分配内存块，即：

- 该内存那块的起始地址先前是由先前对分配方法的调用返回的，如：`alloc`
- 内存块随后没有被释放，通过传递给诸如`dealloc`之类的释放。传递给返回非空指针的重新分配方法。

example：

```rust
use std::alloc::{GlobalAlloc,Layout,alloc};
use std::ptr::null_mut;

struct A;
unsafe impl GlobalAlloc for A {
    unsafe fn alloc(&self, _layout:Layout)->*mut u8{null_mut}  // 分配
    unsafe fn dealloc(&self, _ptr:*mut u8,_layout:Layout){}    // 回收
}

#[global_allocator]
static A:A=A;
fn main(){
    unsafe{
 		assert!(alloc(Layout::new::<u32>()).is_null())       
    }
}
```

### Safety

GlobalAlloc trait 是 unsafe的，遵循的约定：

- 如果全局分配器展开，这是未定义的行为。有可能引起panic，违反内存安全的原则
- `Layout`  查询和计算必须正确

## Methods

### alloc

```rust
unsafe fn alloc(&self,layout:Layout)->*mut u8
```

按照给定的布局分配内存。

分会指向新内存的指针，或者 `null`指示分配失败

#### safety

alloc 是unsafe对的，若调用者不能确保 Layout具有非零大小，则会导致未定义的行为。

#### Errors

返回空指针表示内存已耗尽，或`Layout` 不符合此分配器大小或对齐约束。

### dealloc

```rust
unsafe fn dealloc(
    &self,
    ptr:*mut u8,
    layout:Layout
)
```

使用给定的`layout` 在`ptr` 分配 内存块

#### safety

该函数是unsafe的，调用者不能确保以下的条件：

- `ptr` 表示当前通过此分配器 分配的内存区块
- `layout` 必须与用于分配该内存区块的布局相同。

### alloc_zeroed

```rust
unsafe fn alloc_zeroed(&self,layout:Layout)-> *mut u8{}
```

类似于`alloc` ,在返回之前将内存置零。

#### safety

该函数是unsafe，类似于alloc 不同之处则是保证已经分配的内存即将初始化。

#### Errors

内存消耗完的时候 返回一个`null`的指针 或者layout的分配 对齐方式不符合分配器。

### realloc

```rust
unsafe fn realloc(
	&self,
    ptr:*mut u8,
    layout:Layout,
    new_size:usize
)->*mut u8
```

压缩或扩展内存块到给定的`new_size`  该块由 `ptr`和`layout`提供。

若返回空指针，则ptr引用的该内存块的所有权已经转移到了分配器，视为不可用。新的内存块由`layout` 但是 size 已经变为了 `new_size`

若为`null` 则内存的所有权并未转移，且该内存块未被更改

#### safety

该函数是unsafe的， 即：

- `ptr` 必须由当前分配器所分配。
- `layout` 必须与用于分配的内存块相同
- `new_size` 必须大于零
- `new_size` 当四舍五入到`layout.align（）`的最接近倍数时，一定不能溢出（即，四舍五入的值必须小于`usize :: MAX`）。 

#### Errors

如果新布局不符合分配器的大小和对齐约束，或者如果重新分配否则失败，则返回null。















