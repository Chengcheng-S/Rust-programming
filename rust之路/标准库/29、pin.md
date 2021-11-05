# std::pin

将数据固定到其在内存中的位置

> 构建自引用结构，因为移动带有指向自身指针的对象将使它们失效，这可能导致未定义的行为。

`Pin<P>`确保任何P类型指针的指针对象在内存中有一个稳定的位置，这意味着它不能被移到其他地方，它的内存在被丢弃之前不能被释放。

> 默认情况下，Rust中的所有类型都是可移动的。Rust允许按值传递所有类型，而常见的智能指针类型（如Box<T>和&mut T T）允许替换和移动它们所包含的值：可以移出Box<T>，也可以使用mem:：swap。Pin<P>封装了一个P类型的指针，因此Pin<Box<T>>的功能非常类似于一个常规的Box<T>：当Pin<Box<T>>被丢弃时，它的内容也会被释放，内存也会被释放。类似地，Pin<&mut T >与&mut T 非常相似。但是，Pin<P>不允许客户端实际获得Box<T>或&mut T到固定数据，这意味着不能使用mem:：swap:
>
> ```rust
> use std::pin::Pin;
> fn swap_pins<T>(x:Pin<&mut T>,y:Pin<&mut T>){
>     
> }
> ```

Pin<P>并没有改变Rust编译器认为所有类型都是可移动的这一事实。对于任何T，mem:：swap都是可调用的。相反，Pin<P>通过使调用需要&mut T 的方法（如mem:：swap）而防止移动某些值（由Pin<P>包装的指针指向）

> Pin<P>可用于包装任何P类型的指针，因此它与Deref和DerefMut交互。其中P:Deref的Pin<P>应该被视为指向pinned P:：Target的“P样式指针”——因此，Pin<Box<T>>是指向固定T的自有指针，Pin<Rc<T>>是指向固定T的引用计数指针。为了正确起见，Pin<P>依赖于Deref和DerefMut的实现，而不是移出它们的自参数，并且只返回一次在固定指针上调用固定数据时指向这些数据的指针。

### Unpin

许多类型总是可以自由移动，即使是固定的，因为它们不依赖于有一个稳定的地址。这包括所有基本类型（如bool、i32和references）以及仅由这些类型组成的类型。

对于T:Unpin，Pin<Box<T>>和Box<T>功能相同，Pin<mut T>和&mut T T也一样。

注：固定和取消固定只影响指向类型P:：Target的指针类型，而不影响封装在Pin<P>中的指针类型P本身。

例如：Box<T>是否取消固定对Pin<Box<T>>的行为没有影响（这里，T是指向的类型）。

### Drop guarantee

固定的目的是能够依赖于一些数据在内存中的位置。要实现这一点，不仅要限制移动数据，还要限制用于存储数据的内存的重新分配、重新调整用途或以其他方式使其无效。具体地说，对于固定的数据，必须保持不变，即从固定到调用drop，其内存不会失效或重新调整用途。只有当drop返回或panic时，内存才能被重用。

注意，这个保证并不意味着内存不会泄漏！完全可以不调用pined元素上的drop

### Drop implement

drop函数接受&mut self，但即使类型以前被固定住，也会调用它！

这不会在安全代码中引起问题，因为实现依赖于固定的类型需要不安全的代码，但是请注意，决定在类型中使用固定，也会对Drop实现产生影响。如果类型的元素已经锁定，则必须将Drop视为隐式获取Pin<&mut Self>

### Pin is not Stcurtural for field

如果确定某个字段没有结构固定，那么需要确保的是，永远不会创建对该字段的固定引用。

没有结构固定的字段可能有一个将Pin<&mut Struct>转换为mut Field的投影方法

```rust
impl Struct {
    fn pin_get_field(self: Pin<&mut Self>) -> &mut Field {
        // This is okay because `field` is never considered pinned.
        unsafe { &mut self.get_unchecked_mut().field }
    }
}
```

即使字段类型不是取消固定，也可以对Struct执行取消固定。当没有创建`Pin<&mut Field>`时，该类型对固定的看法与此无关。

### Pin is Stcurtural for field

- 只有当所有结构字段都取消固定时，结构才必须取消固定。这是默认值，但Unpin是一个安全特性，（注，添加一个投影操作需要不安全的代码，因此取消固定是一个安全特性这一事实并不违背这样一个原则：如果使用不安全的代码，则只需担心任何这些问题。）
- 结构的析构函数不能将结构字段移出其参数。但结构（以及它的字段）以前可能已被固定。必须保证不会在Drop实现中移动字段。且自定义的结构体不能是`#[repr（packated）]`。
- 必须确保遵守Drop保证：一旦您的结构被固定，包含内容的内存不会在不调用内容的析构函数的情况下被覆盖或释放。
- 类型被固定时，不能提供任何其他可能导致数据移出结构字段的操作。SSS