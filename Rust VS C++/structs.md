## structs

### C++

C++中的类和结构从执行的角度来看大体上是相同的，他们都包含字段，并且都可以将方法附加到类

```c++
class Foo{
pubilc:
	//Mthods 公开
	double calculateResule();
protected:
    //Elements 只对子类可见
	virtual double doOperation(double lhs,double rhs);
private:
    // elements 自对自己可见
    bool debug_;
};
```

默认的访问级别是public for struct，private for class。关于模板的一些规则只适用于类。

类还可以使用访问说明符从基类继承。因此，一个类在从另一个类派生时可以指定public、protected或private，这取决于它是否希望这些方法对调用方或子类可见。

```rust
class Size {
public:
  Size(int width, int height);
  int width_;
  int height_;
  int area() const;
};
```

在`.cpp`文件中，可以实现构造函数和方法：

```c++
Size::Size(int width, int height) : width_(width), height_(height) {}
int Size::area() { return width_ * height_; }
```

### Rust

Rust 只有结构体，结构由一个定义(指定字段)以及impl部分(绑定其实现)组成，

```rust
struct Size {
  pub width: i32;
  pub height: i32;
}
```

```rust
impl Size {
  pub fn new(width: i32, height: i32) -> Size {
    Size { width: width, height: height, }
  }
  pub fn area(&self) -> i32 {
    self.width * self.height
  }
}
```

自关键字与C++使用的方式相同，作为调用函数的结构的引用。如果一个函数修改了结构，它必须说&mut self，这表示函数修改了结构。

Rust中并没有继承，结构可以实现零个或多个特征。

## Destructors

C++的析构函数是当对象超出范围或被删除时调用的一种特殊方法。

```c++
class MyClass{
pubilc:
	MyClass(): someMember_(new Resource()){}
	~MyClass(){
		delete someMember_;
		
	}
privete:
	Resource *someMember_;
}
```

在C++中，当对象即将被销毁时，可以声明类析构函数，若自定义的类继承自另一个类，则必须使用虚拟析构函数，以防调用方调用基类上的delete。

由于Rust不进行继承，也没有构造函数，因此清理的方式不同且更简单。实现了Drop trait而不是析构函数。

```rust
impl Drop for A{
	fn drop(&mut self){
		println!("A dropping");
	}
}
```

如果实现了这个trait，那么编译器知道在销毁结构之前调用drop（）函数。就这么简单。

有时，在结构超出作用域之前，可能会有明确的理由删除它。也许应该尽快释放变量所持有的资源，以释放处于争用状态的资源。

### 访问规则

C++类可以使用公共、私有和受保护的关键字隐藏或显示方法和成员到任何其他类，或将其自身继承的事物隐藏或显示。

- `pubilc` ——  可以被类内部或外部的任何代码所引用
- `private`——  只能由类内部的代码使用。甚至子类也不能访问这些成员。
- `protected`——  可以由类内部的代码和子类使用。

类可以将另一个函数或类指定为友元，该友元可以访问类的私有成员和受保护成员

相比于C++ Rust则没有那么的繁琐，关键字`pub` 表明这个对象可以被外部所访问到。若没有`pub` 其只能在模块或子模块中可见。

### Functions

方法可以绑定到impl块中：

```rust
impl Shape {
  pub fn new(width: u32, height: u32) -> Shape {
    Shape { width, height }
  }
  pub fn area(&self) -> i32 {
    self.width * self.height
  }
  pub fn set(&mut self, width: i32, height: i32) {
    self.width = width;
    self.height = height;
  }
}
```

以&self/&mut self参数开头的函数绑定到实例。那些没有被绑定到类型。因此new()函数可以称为Shape::new()

```rust
let shape = Shape::new(100, 100);
let area = shape.area();
```

### 静态方法

静态方法指示impl块中没有`&slef`或`&mut self`作为第一个参数的方法。

```rust
impl Circle {
   fn pi() -> f64 { std::f64::consts:PI }
}
//...
let pi = Circle::pi();
```

其并没有绑定到类型示例上，而是直接绑定到了类型上。

### Traits

C++允许一个类从另一个类继承。一般来说，这是一个有用的特性，尽管如果实现多重继承，尤其是可怕的菱形模式，它可能会变得相当复杂。

Rust根本没有类，也就没有类继承这么一言。但是自定义的结构体可以实现部分类的特性。

```rust
trait HasCircumference {
  fn circumference(&self) -> f64;
}
```

### Lifetimes

在C++中，对象从被构造的时刻到被销毁的时刻而存在。

如果在堆栈上声明对象，则该生存期是隐式的。当对象进出范围时，它将被创建/销毁。

如果自定义的对象是另一个对象的成员，则它也是隐式的生命周期在包含对象内，并且包含对象中其他成员的声明顺序也是隐式的。

若使用`new`分配对象，那么何时删除就由开发者决定(PS:忘记删除的话，对程序稳定性影响也是很大的。) 

C++鼓励使用管理对象生命周期的智能指针，将其绑定到智能指针本身的隐式生存期——当智能指针被破坏时，它删除所保存的指针。一种更复杂的智能指针允许同一指针的多个实例同时存在，并使用引用计数，以便在销毁最后一个智能指针时销毁该指针。

### Rust中的生命周期

Rust确实关心对象的生命周期，并跟踪它们以确保不能引用不再存在的对象。

编译器还实现了一个借用检查器，该检查器跟踪对对象的引用，以确保：

- 引用的保存时间不超过他所引用对象的生命周期
- 一次只能有一个可变引用，不能与不可变引用并发。这是为了防止数据竞争

如果编译器发现代码违反其规则，它将生成编译错误。

```
struct Incrementor<'a> {
  value: &'a mut i32
}
impl <'a> Incrementor<'a> {
  pub fn increment(&mut self) -> i32 {
    *self.value += 1;
    *self.value
  }
}
```

大多数情况下，Rust的生命周期是可以省略的，

基本上，它假设在将引用传递给函数时，引用的生存期隐式地长于函数本身，因此不需要进行注释。

Rust生命周期省略的规则：

- 每一个引用参数都拥有自己的生命周期参数
- 当存在多个声明周期时，其中一个是`&self` 或`&mut self` self的生命周期会被赋予所有输出声明周期参数。
- 当有一个生命周期时，这个声明周期会被赋予所有输出生命周期参数。

### 注释

Rust注释类似于C++，除了它们可能包含Unicode，因为.RS文件是UTF-8编码的：

```rust
/*
*/

// do something
```

此外，任何`///` 所注释的内容都可以通过`rustdoc` 工具进行解析，生成文档

在项目上运行cargo doc将导致从文件中带注释的注释生成HTML文档

## 生命周期、引用、借用

当一个对象赋给Rust中的一个变量时，被认为是绑定了它。变量“拥有”对象，只要它在作用域内，当它超出作用域就会被销毁。

```rust
{
  let v1 = vec![1, 2, 3, 4]; // Vec is created
  ...
} // v1 goes out of scope, Vec is dropped
```

所以变量是有作用域的，作用域是影响变量生存期的约束。在范围之外，变量无效。

变量必须绑定到某个对象，如果未使用某种值初始化变量，则不能使用该变量。

在C++中声明变量和不做任何事情都是非常有效的。或者有条件地对变量做一些使编译器混淆的操作，这样它只会生成一个警告。

如果变量未初始化，Rust编译器将抛出错误，而不是警告。如果您声明了一个变量而最终没有使用它，它还会发出警告。

### 引用

编译器跟踪对象的所有权。如果将一个变量赋给另一个变量，则表示对象的所有权已转移给受让人。原始变量无效，如果使用，编译器将生成错误。















