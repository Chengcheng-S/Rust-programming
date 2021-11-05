# 高级trait

### 关联类型在trait定义中指定占位符类型

**关联类型**是一种将**类型占位符与triat相关联**的方式。这样的trait的方法签名中就可以使用这些占位符类型，trait 的实现者会针对**特定的实现在这个类型的位置**指定相应的**具体类型**。如此就可以定义一个使用多种类型的trait。

#### 一个例子

标准库中的**Iterator**，中有一个叫做Item的关联类型来替代遍历的值的类型，定义如下

```rust
pub trait Iterator{
	type Item;
	fn next(&mut self)->Option<Self::Item>
}
```

Item 是一个**占位类性**，同时next方法定义表明他返回**Option<Self::Item>**类型的值，这个trait的**实现者**会**指定Item的具体类型**，然而不管实现者为何种类型。next方法都会返回一个包含了此具体类型值得**Option**

关联类型看起来像一个类似泛型的概念，因为它允许定义一个函数而不指定其可以处理的类型。

Counter 关联类型实现

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--

```

泛型实现

```rust
pub trait Iterator<T>{
	fn next(&mut self)->Option<T>;
}
```

使用**泛型**和**关联类型**的**区别**：使用**泛型**时，需要在**每一个实现**中**标注类型。** 当trait有泛型参数时，可以**多次实现**这个trait，每次需要**改变**泛型**参数的具体类型**当使用 `Counter` 的 `next` 方法时，必须提供类型注解来表明希望使用 `Iterator` 的哪一个实现。

**关联类型** 则**无需标注**类型，因为**不**能**多次实现**这个triat，我们只能选择一次 `Item` 会是什么类型，因为只能有一个 `impl Iterator for Counter`。当调用 `Counter` 的 `next` 时不必每次指定我们需要 `u32` 值的迭代器。

### 默认泛型类型参数

当使用泛型类型参数时，可以为**泛型指定**一个**默认的具体类型**，如果默认类型就足够的话，这消除了为具体类型实现trait的需要，为泛型类型**指定默认语法**是在**声明泛型类型**时使用。

**运算符重载**：在特定情况下，自定义运算符行为的操作

Rust 并**不允**许创建**自定义**运算符或**重载**任意运算符，但是在**std::ops**中所列出的运算符和相应的triat可以通过实现运算符相关的**trait**来**重载。** 

```rust
use std::ops::Add;

#[derive(Debug,PartialEq)]

struct Point{
    x:i32,
    y:i32,
}
impl Add for Point{
    type Output=Point;
    fn add(self,other:Point)->Point{
        Point{
            x:self.x+other.x,
            y:self.y+other.y,
        }
    }
}


fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}

```

实现**Add trait**重载Point实例的**+**运算符

`add` 方法将两个 `Point` 实例的 `x` 值和 `y` 值分别相加来创建一个新的 `Point`。`Add` trait 有一个叫做 **`Output` 的关联类型，它用来决定 `add` 方法的返回值类型**。

默认泛型类型位于Add trait中

```rust
#![allow(unused)]
fn main() {
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;   // Output  关联类型
	}
}
```

此程序具有一种方法和关联类型的特征， `RHS=Self`即**默认类型参数**，**泛型参数RHS** 定义了add方法中的**rhs参数的类型**，如果在实现的过程中未指定rhs的参数类型，那么就会使用默认的Self

一个特例

```rust

#![allow(unused_variables)]
fn main() {
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
}
```

为了将Millimeters和Meter相加，我们指定impl Add<Meters>来设置RHS的值，而没有指定Self

默认参数的场景：

-  扩展一个类型而不破坏现有代码
- 允许大部分用户都不需要的特定场合进行自定义。 

### 完全限定语法与消除歧义：调用该相同名称的方法

Rust 既不能避免一个 trait 与另一个 trait 拥有**相同名称的方法**，也不能阻止为**同一类型**同时**实现这两个 trait**。甚至直接**在类型上实现开始已经有的同名方法**也是可能的。



```rust
trait Pilot{
	fn fly(&self);
}
trait Wizarf{
	fn fly(&self);
}

struct Human;
impl Pilot for Human{
	fn fly(&self){
		println!("this is pilot for human");
	}
}

impl Wizard for Human{
	fn fly(&self){
		println!("this is wizard for human");
	}
}
impl Human{
   fn fly(&self){
       println!("human method");
    } 	
}
```

两个 trait 定义为拥有 `fly` 方法，并在直接定义有 `fly` 方法的 `Human` 类型上实现这两个 trait

```rust
fn main(){
	let person =Human;
	person.fly();//调用human的fly方法
}    //  结果为：human method
```

根据运行结果表明了Rust 直接实现了Human上的fly方法。

那么问题来了 如何调用Pilot和Wizard中的fly呢？

```rust
fn main(){
	let person=Human;
	person.fly();   //  同上 不赘述
 	Pilot::fly(&person);  //  pilot上的fly方法
    Wizard::fly(&person);  //  wizard 上的fly方法
}
```

运行之后依次打印出方法中的信息

**因为 `fly` 方法获取一个 `self` 参数，如果有两个 类型 都实现了同一 trait，Rust 可以根据 `self` 的类型计算出应该使用哪一个 trait 实现。**

如要是参数不是self  而是一个关联函数呢？

此时Rust是无法完成推到的，需要使用**完全限定的语法**

```rust
<Type as Trait>::method(receiver_if_method,next_age);
```



```rust
trait Animal {
    fn message(&self);
    fn speak() -> String;
}

trait Human {
    fn message(&self);
    fn speak() -> String;
}

struct Ani {}

impl Animal for Ani {
    fn message(&self) {
        println!("这是动物");
    }
    fn speak() -> String {
        "这是动物特征".to_string()
    }
}

struct per{}

impl Human for per{
    fn message(&self) {
        println!("这是人类");
    }
    fn speak() -> String {
        "这是人的特征".to_string()
    }
}

impl Human for Ani{
    fn message(&self) {
        println!("这是为动物实现人的特征");
    }
    fn speak() -> String {
        "这是人+动物的第一特征".to_string()
    }
}


impl Animal for per {
    fn message(&self) {
        println!("这是认类实现动物特征");
    }
    fn speak() -> String {
        "这是为认类实现动物特征特征".to_string()
    }
}



fn main() {
    let pe=per{};
    let an=Ani{};

    // 为两个结构体实现Human特征

    Human::message(&pe);
    Human::message(&an);
    println!("{}",<per as Human>::speak());
    <Ani as Human>::speak();

    println!("{}",<per as Animal>::speak());
}

```

对于关联函数，其没有一个**recevier**，故只会有其他参数的列表，可以选择在任**何函数或者方法调用处使用完全限定语法**，

> 然而，允许省略任何 Rust 能够从程序中的其他信息中计算出的部分。只有当存在多个同名实现而 Rust 需要帮助以便知道我们希望调用哪个实现时，才需要使用这个较为冗长的语法。

### 父trait用于在另一个trait中使用某个trait功能

在某个trait中使用另一个trait的功能，此种情况下，需要能够依赖相关的trait也被实现，这个所需要实现的trait就是所实现的父(超)trait

#### 使用newtype模拟用以在外部类型上实现外部trait

**孤儿规则**：只要**trait**或**类型**对于**当前crate**是**本地**的话就可以再此类型上**实现**该trait。

绕开这一原则的方法则是使用**newtype**模式，其涉及到一个**元组结构体**中创建新类型。

该元组结构体带有一个**字段**作为希望实现trait的类型的简单封装，接着这个封装类型对于crate本地的，这样就可以在这个封装上实现trait。

**使用这个模式没有运行时性能惩罚，这个封装类型在编译时就被省略了。**

##### 例

在 `Vec<T>` 上实现 `Display`，而孤儿规则阻止我们直接这么做，因为 `Display` trait 和 `Vec<T>` 都定义于我们的 crate 之外。

```rust
use std::fmt;
strcut Wrapper(Vec<String>);
impl fmt::Display for Wrapper{
	fn fmt(&self,f:&mut fmt::Formatter)->fmt::Result{
			write!(f,"[{}]",self.0.join(", "))	
	}
}   //  为wrapper实现了Display

fn main(){
    let w= Wrapper(vec![String::from("hello"),String::from("world")]);
    println!("w={}",w);
}
```

Display的实现使用了**self.0** 来访问其内部的**Vec<T>**，因为Wrapper是**元组结构体**，而Vec<T>是结构体总体位于**索引0**的，然后就可以为Wrpper实现Display功能了。

缺点：Wrapper是一个**新的类型**，其**没有**定义于其**值之上**的方法，必须在Wrapper上实现**Vec<T>**的所有方法，如此就可以代理到了self.0上，**这就允许我们完全像 `Vec<T>` 那样对待 `Wrapper`**

















