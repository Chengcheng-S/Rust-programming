### std::any::type_name

```rust
pub fn type_name<T>()-> &'static str
where 
	T: ?sized
```

以字符串切片的形式返回类型的名称

```rust
assert_eq!(
	std::any::type_name::<Option<String>>(),
    "core::option::Option<alloc::string::String>",
);
```

