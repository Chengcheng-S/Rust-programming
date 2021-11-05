### std::any::type_name_of_val

```rust
pub fn type_name_of_val<T>(_val:T)->&'static str
where
	T:?Sized
```

以字符切片指针的形式返回类型的名，等同与`type_name::<T>()`。

```rust
#![feature(type_name_of_val)]
use std::any::type_name_of_val;

fn main() {
    let x=1;
    println!("{}",type_name_of_val(&x));

    let y=1.0;
    println!("{}",type_name_of_val(&y));
}
```

