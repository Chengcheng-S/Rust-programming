## std::allLoc::AllocRef

```rust
pub unsafe trait AllocRef{
	fn alloc(&mut self,layout:Layout)->Result<NonNull<[u8]>,AllocErr>;
    unsafe fn dealloc(&mut self,ptr:NonNull<u8>,layout:Layout);
    fn alloc_zeroed(
    	&mut self,
        layout:Layout
    )->Result<NonNull<u8>,AllocErr> {}
    unsafe fn grow(
    	&mut self,
        ptr: NonNull<u8>,
        layout:Layout,
        new_size:usize
    )->Result<NonNull<[u8]>,AllocErr>{}
    
    unsafe fn grow_zeroed(
    	&mut self,
        ptr:NonNull<u8>
        layout:Layuot,
        new_size:usize
    )->Result<NonNull<[u8]>,AllocErr>{}
    
    unsafe fn shrink(
    	&mut self,
        ptr: NonNull<u8>,
        layout:Layout,
        new_size:usize
    )->Result<NonNull<[u8]>,AllocErr>{}
    fn by_ref(&mut self)->&mut Self
}
```

实现AllocRef 可以分配，增长，缩小和取消分配通过Layout描述的任意数据块。

`AllocRef` 设计为ZSTs，引用，智能指针 所实现，无法移动类似于`MyAlloc[u8;N]`这样的分配器，而不更新指向分配的内存指针。

不同于`GlobalAlloc` AllocRef 允许zero_size 分配，若基础分配器不支持此功能,或返回空指针。必须被实现捕获。

### 当前分配的内存

某些方法要求当前通过分配器分配内存块，即：

- 该内存块的起始地址先前通过 alloc， grow  或者 shrink 方法返回.
- 内存块随后没有被重新分配，其中，要么通过传递给dealloc 来直接重新分配存储块，要么通过传递返回OK的 grow 或shrink 来对其进行更改。若grow 或shrink返回Err，则传递的指针保持有效。

### Memory fitting

某些方法要求布局适合存储块。(等效于 内存块适合于布局)，即：

- 必须以`layout.align()`相同的方式分配该块。
- 提供的`layout.size()` 必须在`min..=max` 之间，
  - `min`  最近用于分配块的布局大小
  - `max` 由`grow` `shrink` `alloc` 返回的最新块的实际大小。

### safety

- 从分配器分配的内存块必须指向有效内存，并且保持有效性，直到删除实例或者克隆
- clone 或 move 不得使得这个分配器所指向的内存块无效，clone 分配器的行为必须和之前的分配器一致。
- 指向当前分配的内存块的任何指针可以传递到分配器的其他方法。

## methods

### alloc

```rust
fn alloc(&mut self,layout:Layout)->Result<NonNull<[u8]>,AllocErr>
```

  分配一个内存块。如果成功的话返回满足layout的大小和对其保证的`NonNull<[u8]>` 

返回块的大小可能大于`layout.size()` 指定的大小，并且可能已初始化或未初始化器内容。 `[NonNull<[u8]>]`:NonNull

#### Errors

内存耗尽，或者layout不符合分配器的大小对齐约束。

### dealloc

```rust
unsafe fn dealloc(&mut self,ptr:NonNull<u8>,layout:Layout)
```

取消分配ptr 引用的内存

#### safety

- `ptr`  必须表示当前通过这个分配器分配的内存块。
- `layout` 必须适合内存块

## provide methods

### alloc_zeroed

```rust
fn alloc_zeroed(&mut self,layout:Layout)->Result<NonNull<[u8]>,AllocRef>
```

类似于 alloc 确保返回的内存是零初始化的。

### Errors

内存耗尽，或者layout不符合分配器的大小或对齐方式。

### grow

```rust
unsafe fn grow(
	&mut self,
	new_size:usize,
    ptr:NonNull<u8>,
    layout:Layout
)->Result<NonNull<[u8]>,AllocErr>
```

扩展内存块，

返回一个新的`NonNull<[u8]>` 包含指针和已经分配的内存的实际大小。该指针适用于保存由`new_layout` 描述的数据。 分配器可以扩展由ptr引用的分配以适应新的布局。

Ok： 已将ptr引用的内存块的所有权转移到此分配器，内存(可能)释放，应该被认为是不可用的，除非通过此方法的返回值///重新将其转移回调用方。

Err： 存储块的所有权并未转移到分配器，并且该存储块的内容未更改。

#### safety

- ptr   必须表示通过此分配器分配的一块内存
- `layout` 必须适合这个内存块(`new_size`参数不必适合)
- `new_size` 必须小于`layout.size()`
- `new_layout.size()` 必须大于或等于 `old_layout.size()`

###   grow_zeroed

```rust
unsafe fn grow_zeroed(
	&mut self,
    ptr:NonNull<u8>,
    new_size:usize,
    layout:Layout
)->Result<NonNull<[u8]>,AllocErr>
```

类似于grow， 确保在新内容返回之前将其设置为零。

成功调用`grow_zeroed`之后，存储块将包含以下内容：

- 字节 `0..layout.size()`  从原始分配器中保留
- 字节`layout.size()..old_size` 将被保留或清零 取决于分配器的实现， `odl_size` 表示在`grow_zeroed` 调用之前内存块的大小，或许大于分配时最初请求的大小
- 字节`old_size..new_size`  清零， `new_size` 表示由`grow_zeroed` 调用所返回内存块的大小。

#### safety

- `ptr` 通过当前分配器分配一块内存
- `layout` 必须适应内存块(new_sized 不必)
- `old_layout` 必须适应内存块， new_layout 不必
- `new_size`  必须大于或等于 old_layout 

#### Errors

 内存分配完 ， layout 不符合分配器大小或对齐方式

### shrink

```rust
unsafe fn shrink(
	&mut self,
    ptr:NonNull<u8>,
    layout:Layout,
    new_size:usize
)->Result<NonNull<[u8]>,AllocErr>
```

压缩内存块

返回一个新的`NonNull<[u8]>` 包含指针和已经分配的内存的实际大小。该指针适用于保存由`new_layout` 描述的数据。 分配器可以扩展由ptr引用的分配以适应新的布局。

Ok： 已将ptr引用的内存块的所有权转移到此分配器，内存(可能)释放，应该被认为是不可用的，除非通过此方法的返回值///重新将其转移回调用方。

Err： 存储块的所有权并未转移到分配器，并且该存储块的内容未更改。

### by_ref

```rust
fn by_ref(&mut self)->&mut Self
```

为此AllocRef实例创建“按引用”适配器。返回的适配器也实现AllocRef，并且将简单地借用它。