# std::todo!

```rust
macro_rules! todo {
    () => { ... };
    ($($arg:tt)+) => { ... };
}
```

表示未完成的代码。

> 若正在进行原型设计，并且只希望对代码进行类型检查，那么这将非常有用。
>
> 和`unimplemented!`的区别   `todo!` 传达了稍后实现该功能的意图，并且消息“尚未实现”，
>
> `unimplemented!` 没有任何此类要求。其消息是“未实现”。还有一些IDE会标记todo！

### panics

经常会panic

```rust
trait A{
    fn bar(&self);
    fn baz(&self);
}
struct B;
impl A for B{
    fn bar(&self){}
    fn baz(&self){
        todo!();
    }
}
```

