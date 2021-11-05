### 使用未初始化的内容来创建rc

使用未初始化的内容构造了rc，使用0字节填充了内存

```rust
#![feature(new_uninit)]
#![feature(get_mut_unchecked)]
use std::rc::Rc;

let mut a = Rc<u32>::new_uninit();

let a = unsafe{
    Rc::get_mut_unchecked(&mut a).as_mut_ptr().write(5);
    a.assume_init();
}
```

```rust
#![feature(new_uninit)]
use std::rc::Rc;

let zero=Rc<u32>::new_zeroed();
let zero=unsafe{
	zero.assume_init();
}
```

### 构造一个pin<rc<T>>

若T未实现`unpin`则value将被固定在内存中的某个位置而无法移动

### try_unwrap

若rc中的值为强引用则返回值，否则 `Err` ,若存在未完成的弱引用，此操作也会成功

```rust
use std::rc::Rc;
let x =Rc::new(4);
assert_eq!(Rc::try_unwrap(x),Ok(4));

let y = Rc::new(10);
let _z = Rc::clone(&y);
assert_eq!(*Rc::try_unwrap(y).unwrap_err(),10);
```

### into_raw

消耗`Rc`指针，返回包装的指针，为避免内存泄漏，必须使用`[Rc::from_raw][from_raw]`将指针转换回`Rc`

```rust
pub fn into_raw(this:Self)->*const T{
	let ptr =Self::as_ptr(&this);
    mem::forget(this);
    ptr
}
```

eg:

```rust
use std::rc::Rc;

let x =Rc:new("rust".to_owned);
let x_ptr= Rc::into_raw(x);
assert_eq!(unsafe{&*x_ptr},"rust");
```

### as_ptr

返回数据的原始指针，计数不会受到任何影响，并且不会消耗`Rc`。只要`Rc`中有强引用的计数，指针就有效

```rust
use std::rc:Rc;

let x =Rc::new("rust".to_owned());
let y = Rc::clone(&x);
let x_ptr- Rc::as_ptr(&x);
assert_eq!(x_ptr,Rc::as_ptr(&y));
```

### from_raw

从原始指针构造一个`Rc<T>`之前必须通过`into_raw` 返回原始指针，unsafe

```rust
use std::rc::Rc;

let x = Rc::new("rust".to_owned());
let y = Rc::into_raw(x);

unsafe{
    let x =Rc::from_raw(y);
    assert_eq!(&*x,"rust");
}
// x 离开作用域时， 内存释放，y就会别为悬垂指针
```

### downgrade

创建Weak指针

```rust
use std::rc::Rc;

let a =Rc::new(5);
let a_weak = Rc::downgrade(&a);
```

```rust
  #[stable(feature = "rc_weak", since = "1.4.0")]
    pub fn downgrade(this: &Self) -> Weak<T> {
        this.inner().inc_weak();
        // Make sure we do not create a dangling Weak
        debug_assert!(!is_dangling(this.ptr));
        Weak { ptr: this.ptr }
    }
```

###  weak_count

得到weak指针的数量

### strong_count

获取强引用的数量

```rust
use std::rc::Rc;
let a =Rc::new(5);
let _b = Rc::clone(&a);
let c = Rc::strong_cout(&a); //2
```

### get_mut

若没有其他指向该分配的`Rc`或`Weak` 指针，则返回true

如果没有其他指向同一分配的`Rc`或[`Weak`]指针，则返回给定`Rc`的可变引用。否则返回`None` 对共享值进行可变是unsafe的

```rust
use std::rc::Rc;

let mut x = Rc::new(3);
*Rc::get_mut(&mut x).unwrap()=4;
assert_eq!(*x,4);

let _y = Rc::clone(&x);
assert!(Rc::get_mut(&mut x).is_none());
```

### get_mut_unchecked

将可变对的引用返回给定的`Rc`,而不进行检查 unsafe

### make_mut

对给定的`Rc`进行可变引用，如果还有其他指向同一分配的`Rc`指针，则`make_mut`将`clone`内部值分配给新分配以确保唯一所有权，也称为`cow`

若没有其他指向给分配的`Rc`指针，则`weak`指向该发呢配的指针将被取消关联

```rust
use std::rc::Rc;

let mut data=Rc::new(5);    // 不会clone 任何东西
*Rc::make_mut(&mut date)+=1;   // clone inner data
let mut b = Rc::clone(&data);
*Rc::make_mut(&mut data)+=1;
*Rc::make_mut(&mut data)+=1;
*Rc::make_mut(&mut b) *=2;
//  data 和b 都指向了新的内存地址
assert_eq!(*data,8);
assert_eq!(*b,12);
```

对于weak来说 将取消引用

```rust
use std::rc::Rc;

let mut data= Rc::new(75);
let weak =  Rc::downgrade(&data);

assert!(75==*data);
assert!(75=*weak.upgrade().unwrap());

*Rc::make_mut(&mut data) +=1;
assert!(76 == *data);
assert!(weak.upgrad().is_none());
```

### downcst

将`Rc<dyn Any>` 转为具体类型

```rust
pub fn downcast<T:Any>(self) ->Result<Rc<T>,Rc<dyn Any>>{
	if (*self).is::<T>(){
        let ptr = self.ptr.cast::<RcBox<T>>();
        forget(self);
        Ok(Rc::from_inner(prt))
    }else{
        Err(self)
    }
}
```

eg:

```rust
use std::rc:Rc;
use std::any::Any;
fn fa(vlaue : Rc<dyn Any>){
    if let Ok(string) = value.downcast::(String)(){
        println!("String ({}):{}",string.len(),string);
    }
}
let strings ="rustlang".to_string();
fa(strings)
```

### drop

删除`Rc` ，结果减少强引用计数，若强引用计数置零，那么弱引用计数将被丢弃

```rust
use std::rc::Rc;

struct Foo;
impl Drop dor Foo{
    fn drop(&mut self){
        println!("droped")
    }
    
}

let foo = Rc::new(Foo);
let fa = Rc::clone(&foo);

drop(foo);  // 什么也不打印
drop(fa);   // 打印 droped
```

### clone

clone `Rc` 指针，将创建另一个指向相同分配的指针，从而增加strong计数



> Weak`是[`Rc`]的版本，它拥有对///托管分配的非所有者引用。通过调用弱///指针上的[upgrade`]来访问分配，该指针返回[`Option`]`<`[`Rc`]`<T >>。 /// ///因为弱引用不计入所有权，所以它不会///阻止存储在分配中的值被丢弃，并且弱本身本身也不保证仍会保留该值在场。因此，当[`upgrade`] d时，它可以返回[`None`] ///。但是请注意，“弱”引用确实会阻止分配///本身（后备存储）被释放。 /// ///`弱指针对于保持由[`Rc`]管理的分配的临时引用很有用，而不会阻止其内部值被丢弃。它也用于///防止[`Rc`]指针之间的循环引用，因为相互拥有的引用///永远都不允许删除任何[`Rc`]

















