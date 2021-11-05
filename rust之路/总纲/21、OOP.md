# Rust`s OOP

Object-Oriented Proramming(面向对象OOP) 

> 定义：
>
> Object-oriented programs are made up of objects. An *object* packages both data and the procedures that operate on that data. The procedures are typically called *methods* or *operations*.

面向对象的程序是由对象组成的。一个对象包含数据和操作这些数据的过程，这些过程通常呗成为**方法**或**操作**

按照如此定义下，Rust是面向对象的：结构体和枚举包含数据而impl块提供了在结构体和枚举之上的方法。(虽然带有 方法的结构体和枚举并不被称为对象)

## 封装

封装思想：对象的实现细节**不能**被使用对象的程序获取到。唯一与**对象交互**的方式是通过对象提供的**公有API**；使用对象的代码无法深入到对象内部并直接改变数据或者行为。封装使得改变和重构对象的内部时无需改变使用对象的代码。

```rust

#![allow(unused)]
fn main() {
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
}
```

## **继承作为类型系统与代码共享**

**继承**是一种机制，一个对象可以定义为继承另一个对象的定义，这使其可以获得父对象的数据和行为，而无需重新定义。

如果一个语言必须有继承才能被称为面向对象的语言的话，那么Rust就不是面向对象的，无法定义一个结构体继承父结构体的成员和方法。

继承作用： 一、重写代码，一旦为一个类型实现了特定行为，**继承可以对一个不同的类型重用这个实现**。相反 Rust 代码可以使用默认 trait 方法实现来进行共享，

二，表现为子类型可以用于父类型被使用的地方。这也被称为 **多态**，如果多种对象共享特定的属性，则可以相互替代使用。

> 多态: 对于继承来说，这些类型通常是子类。 Rust 则通过泛型来对不同的可能类型进行抽象，并通过 trait bounds 对这些类型所必须提供的内容施加约束。这有时被称为 *bounded parametric*

### 使用trait实现多态

定义一个带有draw方法的Trait Draw

```rust
pub trait Draw{
	fn draw(&self);
}
```

```rust
pub struct Screen{
	pub components:Vec<Box<dyn Draw>>,
}
```

Box<dyn Draw>, 此为一个**triat对象**，它是**Box中任何实现了Draw trait的替身**

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

定义一个run方法，该方法会对其components上的每个组件调用draw方法

这与定义使用了带有 **trait bound 的泛型类型参数的结构体不同。泛型类型参数一次只能替代一个具体类型，而 trait 对象则允许在运行时替代多种具体类型**。

```rust
pub struct Screen<T:Draw>{
	pub component:Vec<T>,
}
impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

这限制了 `Screen` 实例必须拥有一个全是 `Button` 类型或者全是 `TextField` 类型的组件列表。如果只需要同质（相同类型）集合，则倾向于使用泛型和 trait bound，因为其定义会在编译时采用具体类型进行单态化。

通过使用 trait 对象的方法，一个 `Screen` 实例可以存放一个既能包含 `Box<Button>`，也能包含 `Box<TextField>` 的 `Vec<T>`。

trait 实现

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 实际绘制按钮的代码
    }
}
```

库使用者现在可以在他们的 `main` 函数中创建一个 `Screen` 实例。至此可以通过将 `SelectBox` 和 `Button` 放入 `Box<T>` 转变为 trait 对象来增加组件。接着可以调用 `Screen` 的 `run` 方法，它会调用每个组件的 `draw` 方法。

```rust
use  gui::{Screen,Boutton};
fn main(){
	let screen=Screen{
        components:vec![
           Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
             Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };
    screen.run();
}
```

### trait对象执行分发

编译器为每一个被泛型类型参数代替的具体类型生成了非泛型的函数和方法实现。单态化所产生的代码进行 **静态分发**【静态分发发生于编译器在编译时就知晓调用了什么方法的时候】

动态分发：编译器在编译时无法知晓调用了什么方法。在动态分发的情况下，编译器会生成**在运行时确定调用了什么方法的代码**。

当使用trait对象时，Rust必须使用动态分发，编译器无法知晓所有可能用于trait对象代码的类型，所以不确当应该调用哪个类型的哪个方法。**Rust 在运行时使用 trait 对象中的指针来知晓需要调用哪个方法。动态分发也阻止编译器有选择的内联方法代码**

#### trait对象安全

只有**对象安全**的trait才可以组成trait对象。  安全trait的规则

- 返回值类型不为Self
- 方法没有任何和泛型类型参数

Self关键字是要实现trait或方法的**类型的别名**，对象安全对于trait对象是必须的，一旦有了trait对象，就不再知晓**实现该 trait 的具体类型是什么了**

如果triat方法返回具体的Self类型，但是trait对象忘了其正真的类型，**那么方法不可能使用已经忘却的原始具体类型**。

**对于泛型类型参数来说，当使用 trait 时其会放入具体的类型参数：此具体类型变成了实现该 trait 的类型的一部分。当使用 trait 对象时其具体类型被抹去了，故无从得知放入泛型参数类型的类型是什么。**







