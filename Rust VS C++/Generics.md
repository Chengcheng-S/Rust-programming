## Generics/Templates

### C++

C++提供template，使用抽象类型编写泛型代码，然后通过将一个或多个类型替换为具体类来应用在不同的场景。

```c++
temlate<typename T>
inline void debug(const T &v){
	cout <<"the value of object is "<<v<<endl;
    
}
debug(10);
```

class 也可以被模板化

```c++
template<class T>
class stak{
    private:
    	vactir<T> elemnts;
    public:
    	void push(const T &v){}
		T pop(){}
}
Stak<int>()
```

模板是内联的，编译器会在尝试编译之前展开所调用的任何内容

### Generic Functions

Rust中等效于C++中模板称之为泛型，泛化了一个函数或trait，可以处理与条件匹配的不同类型。

```rust
use std::fmt;
fn debug<T>(data: T) where T: fmt::Display {
  println!("The value of object is {}", data);
}
//...
debug(10);
```

#### Generic structs

```rust
struct Stak<T> {
  elements: Vec<T>
}
impl<T> Stak<T> {
  fn new() -> Stak<T> { Stak { elements: Vec::new() } }
  fn push(v: T) {
    //...
  }
  fn pop() -> Option<T> {
    //...
    None
  }
}
//...
let double_stak: Stak<f64> = Stack::new();
```

#### where子句

where子句来对泛型类型必须执行的操作施加约束，以允许将其提供给泛型函数或结构。

```rust
fn compare<T, F>(a: T, b: T, f: F) -> bool 
  where F: FnOnce(T, T) -> bool 
{
  f(a, b)
}
let comparer = |a, b| a < b;
let result = compare(10, 20, comparer);
```

where子句声明函数必须采用两个相同类型的值并返回布尔值。编译器将确保传入的任何闭包都符合该条件.