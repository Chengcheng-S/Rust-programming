## std::borrow::Cow

```rust
pub enum Cow<'a,B>
where 
	B: 'a+ToOwned+ ?Sized,
{
    Borrowed(&'a B),
    Owned(<B as ToOwned>::Owned),
}
```

clone-on-write 智能指针。

