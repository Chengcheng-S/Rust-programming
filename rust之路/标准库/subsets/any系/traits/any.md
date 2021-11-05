## std::any::Any

```rust
pub trait Any: 'static{
	fn type_id(&self)->TypeId
}
```

模拟动态类型triat ， 大多数类型都实现了Any，任何包含非`‘static`的引用类型都不行。

## methods

### type_id

```rust
fn type_id(&self)->TypeId
```

获取`self` 的`TypeId`

```rust
use std::any::{Any,TypeId};
fn  main() {
    assert_eq!(is_str(&1),false);
    assert_eq!(is_str(&"ccc".to_string()),true);
}

fn is_str(s:&dyn Any)->bool{
    TypeId::of::<String>() == s.type_id()
}
```

### 实现; implementations

```rust
impl dyn Any+ 'static
```

```rust
pub fn is<T>(&self)->bool
where 
	T:Any
```

Box类型和T类型相等，则为true

```rust
use std::any::Any;
fn is_str(s:&dyn Any){
    if s.is<String>{
        println!("s'type is string");
    }else{
        println!("can not konw the s's type");
    }
}
```





```rust
pub fn downcast_ref<T>(&self)->Option<&T>
where  T:Any
{	
}
```

若为T类型 则返回Box中的引用，否则 无

example:

```rust
use std::any::Any;

fn f1(s:&dyn Any){
    if let Some(string)=s.downcast_ref::<String>(){
        println!("is a Stirng {}",.len,string.len,string);
    }else{
        println!("Not a string");
    }
}
```



```rust
pub fn downcast_mut<T>(&mut self)->Option<&mut T>
where 
	T:Any,
```

若类型为T 则返回Box中的引用，否则 无

```rust
use std::any::Any;

fn main() {
    let mut a: u32 = 4;
    fa(&mut a);
}

fn fa(s: &mut dyn Any) {
    if let Some(num) = s.downcast_mut::<u32>() {
        *num += 3;
    }
}
```



```rust
impl dyn Any +'static +Send
pub fn is<T>(&self)->bool
where
	 T:Any	 
```

转发到类型为Any的方法。

```rust
use std::any::Any;

fn main() {
    fa(&"rust".to_string());
}

fn fa(s: &(dyn Any + Send)) {
    if s.is::<String>() {
        println!("s is string");
    } else {
        println!("unknow the s type");
    }
}
```



```rust
impl dyn Any+'static +Sync+Send
pub fn is<T>(&self)->bool
where T:Any
```

转发类型为`Any` 方法

```rust
use std::any::Any;

fn is_string(s: &(dyn Any + Send + Sync)) {
    if s.is::<String>() {
        println!("It's a string!");
    } else {
        println!("Not a string...");
    }
}

is_string(&0);
is_string(&"cookie monster".to_string());
```

