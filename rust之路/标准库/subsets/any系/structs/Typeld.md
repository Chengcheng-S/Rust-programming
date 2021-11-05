### std::any::Typeld

```rust
pub struct TypeId{}
```

`TypeId` 全局唯一标识符。

每个`TypeId` 是不透明对象，不允许检查内部的元素，允许简单的操作 clone， comparsion, print。 目前仅适用于`static`

```rust
#[derive(Clone,Copy,Debug,Eq,Hash,Ord,PartialEq,PartialOrd,StructuralEq,StructuralPartialEq)]
```

