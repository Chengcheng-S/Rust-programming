# std::matches

```rust
macro_rules! matches {
    ($expression:expr, $( $pattern:pat )|+ $( if $guard: expr )?) => { ... };
}
```

返回给定表达式是否与任何给定模式匹配

类似于match表达式，模式后面可以有选择地后跟if和一个可以访问模式绑定的名称的保护表达式。

```rust
let foo="f";
matches!(foo,'A'..='Z'|'a'..='z');
```

```rust
let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```

