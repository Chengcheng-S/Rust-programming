# 高级trait

## Default 

默认泛型参数

```rust
trait name<RHS=Self>{
	fn foo(&self,rhs:RHS)->&str;
}
```

第一行定义了一个默认泛型类型，  随后在foo函数中使用了默认的泛型类型。

其可以接收任何的类型的参数，只不过默认的是Self。

## Into /From

隶属`std::convert`

基本形式： `From<T>`, `Into<T>`

### From

对于类型U的对象foo ，如果其实现了From<T>,那么可以通过

```rust
let foo=U::from(bar);
```

bar 即为类型T的对象

#### 例：

```rust
let str="Rust".to_string();
let other_str=String::from("Rust");
```

### Into 

对于一个类型为`U::Into<T>`的对象`foo`, Into提供了一个函数`.into(self)->T`。

调用foo.into()会消耗自己(**转移资源所有权**),**生成类型为T的另一个新对象**。

```rust
fn main() {
    let s="hello".to_string();
    f1(s);
}

fn f1<T:Into<Vec<u8>>>(s:T){
    let bytes= b"hello".to_vec();
    assert_eq!(bytes,s.into());
}
```

#### 一个例子

```rust
struct Person {
    name: String,
}
impl Person {
    fn new<S: Into<String>>(name: S) -> Person {
        Person { name: name.into() }
    }
}
```

参数类型S 是一个泛型参数， S:Into<String> 表示S必须实现了Into<String>，而 `&str` 类型，符合这个要求。因此 `&str` 类型可以直接传进来。而 `String` 本身也是实现了 `Into<String>` 的。当然也可以直接传进来。

name: name.，into()将 `name` 转换成 `String` 类型的另一个对象。当 name 是 `&str` 时，它会转换成 `String` 对象，会做一次字符串的拷贝（内存的申请、复制）。而当 name 本身是 `String` 类型时，`name.into()` 不会做任何转换，代价为零。





调用：

```rust
fn main() {
    let person = Person::new("Herman");
    let person = Person::new("Herman".to_string());
}
```



```rust
impl<'a> From<&'a str> for String {}
impl<T> From<T> for T {}
impl<T, U> Into<U> for T where U: From<T> {}
```



## AsRef /AsMut

`std::convert` 下，`AsRef/AsMut`，它们功能是配合泛型，在执行引用操作的时候，进行**自动类型转换**。



### AsRef 

提供了一个方法 `as_ref()`

对于一个类型T的对象foo ，若T实现了AsRef<U>,那么foo可执行`as_ref()`, 将会得到一个类型为`&U`的新引用

注：

-  不同于Into<T>, AsRef 只是类型转换，对象本身并没有消耗
- T:AsRef<U> 中的T 可以是 owned(资源拥有者)， shared reference(共享引用)， (mutable  reference) 可变引用类型

```rust
fn f2<T:AsRef<str>>(s:T){
	assert_eq!("hello",s.as_ref());
}

let s="hello";
f2(s);

let s = "hello".to_string();
is_hello(s)
```

string和&str 都实现了AsRef<str>

### AsMut

`AsMut<T>` 提供了一个方法 `.as_mut()`。它是 `AsRef<T>` 的可变（mutable）引用版本。

对于一个类型为 `T` 的对象 `foo`，如果 `T` 实现了 `AsMut<U>`，那么，`foo` 可执行 `.as_mut()` 操作，即 `foo.as_mut()`。操作的结果，我们得到了一个类型为 `&mut U` 的可变引用。

注：在转换的过程中，`foo` 会被可变借用。



## Borrow

### Borrow

use std::borrow::Borrow

提供了一个方法 `borrow`,对于一个T类型的值foo，若T实现了了Borrow，那么foo可以执行 `foo.borrow`, 得到一个&U的新引用

`Borrow` 可以认为是 `AsRef` 的严格版本，它对普适引用操作的前后类型之间附加了一些其它限制。

`Borrow` 的前后类型之间要求必须有内部等价性。不具有这个等价性的两个类型之间，不能实现 `Borrow`。

`AsRef` 更通用，更普遍，覆盖类型更多，是 `Borrow` 的**超集**。

```rust
use std::borrow::Borrow;
fn f2<T: Borrow<str>>(s: T) {
    assert_eq!("Hello", s.borrow());
}
let s = "Hello".to_string();
f2(s);
let s = "Hello";
f2(s);
```

### BorrowMut

use std::borrow::BorrowMut

为Borrow的可变版本  拥有方法 borrow_mut(),可以得到 &mut U的可变引用。

在转换过程中，值会被可变借用。

### ToOwned

use  std::borrow::ToOwned;

为Clone的普适版本，提供了to_owned() 方法，用于**类型转换**。

ToOwned 可以从**任何引用类型**实例，生成具有**所有权的类型**实例。

而Clone， 实现了 `Clone` 的类型 `T` 可以从引用状态实例 `&T` 通过 `.clone()` 方法，生成具有所有权的 `T` 的实例。但是它只能由 `&T` 生成 `T`。而对于其它形式的引用，`Clone` 就无能为力了。

## Deref

`Deref` 是 `deref` 操作符 `*` 的 trait，比如 `*v`。

`*v` 操作，是 `&v` 的反向操作，即试图由资源的引用获取到资源的拷贝（如果资源类型实现了 `Copy`），或所有权（资源类型没有实现 `Copy`）。Rust 中，本操作符行为可以重载。这也是 Rust 操作符的基本特点。

### 强制隐式转换

coercion，规则：

一个类型为T的foo，如果 T:Deref<Target=U>,那么foo的某个智能指针或引用，**在应用时自动转为&U**。

Rust 编译器会在做 `*v` 操作的时候，自动先把 `v` 做引用归一化操作，即转换成内部通用引用的形式 `&v`，整个表达式就变成 `*&v`。这里面有两种情况：

1. 将其他的类型指针，转为内部标准形式 &u；
2. 将多重&，简化成&u(通过插入足够的`*`进行解引)

```rust
fn foo(s: &[i32]) {
    // borrow a slice for a second
}
// Vec<T> implements Deref<Target=[T]>
let owned = vec![1, 2, 3];
foo(&owned);
```



```rust
struct Foo;
impl Foo {
    fn foo(&self) { println!("Foo"); }
}
let f = &&Foo;
f.foo();
(&f).foo();
(&&f).foo();
(&&&&&&&&f).foo();
```

`coercion` 的设计，是 Rust 中仅有的类型隐式转换，设计它的目的，是为了简化程序的书写，让代码不至于过于繁琐。

## Cow

Cow 是一个枚举类型 (Clone-on-Write)  `use std::borrow::Cow`, 写时克隆，本质上是一个智能指针。

```rust
pub enum Cow<'a, B> 
where
    B: 'a + ToOwned + 'a + ?Sized, 
 {
    Borrowed(&'a B),    //用于包裹引用
    Owned(<B as ToOwned>::Owned),   //用于包裹所有者
}
```

有两个可选值：

1.  Borrowed， 用于包裹对象的引用(通用引用)
2. Owned， 用于包裹对象的所有者

*以不可变的方式访问借用内容，在需要可变借用或所有权的时候再克隆一份数据*。

提供了：对此对象不可变访问(直接调用此对象的原有的不可变方法)；  若需要修改此对象，或需要获取此对象的所有权的情况，Cow会提供方法做克隆处理，并避免多次重复克隆。

1. Cow<T>, 能直接调用T的不可变方法，因为Cow实现了Deref
2. 在写T时，可以使用to_mut 得到一个具有所有权的值的可变引用
   1. 注： 调用 to_mut 不一定会产生克隆
   2. 在已经获取所有权的情况下，调用to_mut 有效，但不会产生新的克隆
   3. 多次调用一个to_mut 只会产生一次克隆
3. 在需要写T时，使用into_owned()  创建新的拥有所有权的对象，这个过程中内存拷贝和创建新对象：
   1. 若之前Cow是借用状态，调用此操作将执行克隆
   2. 本方法参数是self 类型，会消耗原有的对象，调用之后原先的对象的声明周期就截止了，在Cow上不能多次调用

注： 

- `to_mut` ：就是返回数据的可变引用，如果没有数据的所有权，则复制拥有后再返回可变引用；
- `into_owned` ：获取一个拥有所有权的对象（区别与引用），如果当前是借用，则发生复制，创建新的所有权对象，如果已拥有所有权，则转移至新对象。

```rust
use std::borrow::Cow;
let mut cow: Cow<[_]> = Cow::Owned(vec![1, 2, 3]);
let hello = cow.to_mut();
assert_eq!(hello, &[1, 2, 3]);
```



```rust
use std::borrow::Cow;
let cow: Cow<[_]> = Cow::Owned(vec![1, 2, 3]);
let hello = cow.into_owned();
assert_eq!(vec![1, 2, 3], hello);
```



```rust
use std::borrow::Cow;
fn abs_all(input: &mut Cow<[i32]>) {
    for i in 0..input.len() {
        let v = input[i];
        if v < 0 {
            // clones into a vector the first time (if not already owned)
            input.to_mut()[i] = -v;
        }
    }
}
```

#### 例

过滤输入字符串中所有的空格字符，并返回过滤后的字符串

```rust
fn f3(input:&str)->String{
    let mut buf =Srring::with_capacity(input.len());
    for c in input.chars(){
        if c!=''{
            buf.push(c);
        }
        
    }
    buf
}

```

为何input为&str  而不是String 

使用 `String`， 则外部在调用此函数的时候

1. 若外部字符串是&str，其需要一次克隆，才可以调用该函数
2. 若为String，虽不需要克隆，调用之后 字符串的所有权移至函数的内部，后续无法使用



最坏的情况下 输出input中的内容，如此便会对对象进行拷贝

```rust
use std::borrow::Cow;
fn remove_spaces<'a>(input: &'a str) -> Cow<'a, str> {
    if input.contains(' ') {
        let mut buf = String::with_capacity(input.len());
        for c in input.chars() {
            if c != ' ' {
                buf.push(c);
            }
        }
        return Cow::Owned(buf);
    }
    return Cow::Borrowed(input);
}

```

## 并发原语







### Send 和Sync

std::marker 中的send和sync和多线程有关

标记为`marker tarit`实际上是一种约定，没有方法的定义，也没有关联元素，实现了它的类型必须满足这种约定，

`Send` 和 `Sync` 在大部分情况下（针对 Rust 的基础类型和 std 中的大部分类型），会由编译器自动推导出来。对于不能由编译器自动推导出来的类型，要使它们具有 `Send` 或 `Sync` 的约定，可以由人手动实现。实现的时候，必须使用 `unsafe` 前缀，

定义如下：

如果 `T: Send`，那么将 `T` 传到另一个线程中时（按值传送），不会导致数据竞争或其它不安全情况。

- Send 是对象可以安全发送到另一个执行体中
- Send 使被发送对象可以产生它的线程解耦，防止原线程将此资源释放后，在目标线程中使用出错

如果  `T: Sync`，那么将 `&T` 传到另一个线程中时，不会导致数据竞争或其它不安全情况。

- sync 可以被同时多个执行体访问而不出错
- sync 防止数据竞争

结论：

1. `T: Sync` 意味着 `&T: Send`；
2. `Sync + Copy = Send`；
3. 当 `T: Send` 时，可推导出 `&mut T: Send`；
4. 当 `T: Sync` 时，可推导出 `&mut T: Sync`；
5. 当 `&mut T: Send` 时，不能推导出 `T: Send`；

注： T， &T，&mut T Box<T> 等都是不同的类型

具体的类型：

1. 原始类型（比如： u8, f64），都是 `Sync`，都是 `Copy`，因此都是 `Send`；
2. 只包含原始类型的复合类型，都是 `Sync`，都是 `Copy`，因此都是 `Send`；
3. 当 `T: Sync`，`Box<T>`, `Vec<T>` 等集合类型是 `Sync`；
4. 具有内部可变性的的指针，不是 `Sync` 的，比如 `Cell`, `RefCell`, `UnsafeCell`；
5. `Rc` 不是 `Sync`。因为只要一做 `&Rc<T>` 操作，就会克隆一个新引用，它会以非原子性的方式修改引用计数，所以是不安全的；
6. 被 `Mutex` 和 `RWLock` 锁住的类型 `T: Send`，是 `Sync` 的；
7. 原始指针（`*mut`, `*const`）既不是 `Send` 也不是 `Sync`；

### 同步

>  同步指的是线程之间的协作配合，以共同完成某个任务。在整个过程中，需要注意两个关键点：一是共享资源的访问， 二是访问资源的顺序。

### 等待

-  等待一段时间后，再接着继续执行。通过调用相关的API可以让当前线程暂停执行进入睡眠状态，此时调度器不会调度它执行，等过一段时间后，线程自动进入就绪状态，可以被调度执行，继续从之前睡眠时的地方执行。对应的API   `std::thread::sleep`   `std::thread::sleep_ms`   `std::thread::park_timeout`  `std::thread::park_timeout_ms` 
- 当前线程自己主动放弃当前时间片的调度，让调度器重新选择线程来执行，这样就把运行机会给了别的线程，但是要注意的是，如果别的线程没有更好的理由执行，当然最后执行机会还是它的。  相关API  `std::thread::yeild_now`
- 需要其他线程参与，才能把等待的线程叫醒，否则，线程会一直等待下去。  相关API  `std::thread::JoinHandle::join`  `std::thread::park`   `std::sync::Mutex::lock`

Rust的条件变量就是`std::sync::Condvar`      不管基于什么规则，要触发叫醒这个事件，就肯定是某个条件已经达成了。基于这样的逻辑，在操作系统和编程语言中，引入了一种叫着**条件变量**的东西。    

### 通知

- 通知必然是因为有等待，所以用纸和等待几乎是成对出现的， 如：`std::sync::Condvar::wait`    `std::sync::Condvar::notify_one`    `std::sync::Condvar::notify_all`
- 等待所使用的对象，与通知使用的对象是同一个对象，从而该对象需要在多个线程之间共享
- 除了`Condvar`之外，其实*锁*也是具有自动通知功能的，当持有锁的线程释放锁的时候，等待锁的线程就会自动被唤醒，以抢占锁。
- 通过条件变量和锁，还可以构建更加复杂的自动通知方式，比如`std::sync::Barrier`。
- **通知也可以是1:1的，也可以是1:N的**，`Condvar`可以控制通知一个还是N个，而锁则**不能**控制，只要释放锁，所有等待锁的其他线程都会同时醒来，而不是只有最先等待的线程。

```rust
use std::sync::{Arc,Mutex,Condvar};
use std::thread;


fn main(){
     let pair  = Arc::new((Mutex::new(false),Condvar::new()));

    let pair_two = pair.clone();
    // create new thread
    thread::spawn(move ||{

        let &(ref lock,ref cars) = &*pair_two;

        let mut started = lock.lock().unwrap();

        *started =true;

        cars.notify_one();

        dbg!("the thread run here!!!");
    });

    // wait new thread 

    let &(ref lock, ref cars) = &*pair;

    let mut started = lock.lock().unwrap();

    while !*started{
        println!("brefore wait");

        started = cars.wait(started).unwrap();
        println!("after wait");
    }

}
```



```shell
brefore wait
[src\main.rs:57] "the thread run here!!!" = "the thread run here!!!"
after wait
```

- Mutex 是Rust中的一种锁
- Condvar 需要配合 Mutex 使用， 在Mutex下，Condvar并发才是安全的
- `Mutex::lock`方法返回的是一个`MutexGuard`，在离开作用域的时候，自动销毁，从而自动释放锁，从而避免锁没有释放的问题。
- `Condvar`在等待时，是会释放锁的，被通知唤醒时，会重新获得锁，从而保证并发安全。

### 原子类型

原子类型不需要开发者处理加锁和释放锁的问题，同时支持修改，读取等操作，还具备较高的并发性能，从硬件到操作系统，到各个语言，基本都支持。在标准库`std::sync::atomic`中，查看相关的介绍，包含`AtomicBool`，`AtomicIsize`，`AtomicPtr`，`AtomicUsize`。 `共享资源安全访问`

```rust
use std::thread;
use sync::Arc;
use std::sync::atomic::{AtomicUsize,Ordering}

fn main(){
    let num_one = Arc::new(AtomicUsize::new(3));
    let share_val = num_one.clone();
	
    // create new thread
    thread::spawn(move||{
        println!("the share val:{:?}",share_val.load(Ordering::SeqCst));
    	// change the val    
        share_val.store(7,Ordering::SeqCst);
    }).join().unwrap();
	// output val
    println!("the new val  {:?}",num_one.load(Ordering::SeqCst));   
}
```

### 锁

```rust
use std::sync::{Arc,Mutex,Condvar};
use std::thread;

fn main(){
    let pair  = Arc::new((Mutex::new(false),Condvar::new()));

    let pair_two = pair.clone();
    // create new thread
    thread::spawn(move ||{

        let &(ref lock,ref cars) = &*pair_two;

        let mut started = lock.lock().unwrap();

        *started =true;

        cars.notify_one();

        dbg!("the thread run here!!!");
    });

    // wait new thread 

    let &(ref lock, ref cars) = &*pair;

    let mut started = lock.lock().unwrap();

    while !*started{
        println!("brefore wait");

        started = cars.wait(started).unwrap();
        println!("after wait");
    }
}
```



代码中的`Condvar`就是条件变量，它提供了`wait`方法可以主动让当前线程等待，同时提供了`notify_one`方法，让其他线程唤醒正在等待的线程。 如此达到了顺序控制的目的，在Rust中，`Mutex`是一种独占锁，同一时间只有一个线程能持有这个锁。这种锁会导致所有线程串行起来，这样虽然保证了安全，但效率并不高。

