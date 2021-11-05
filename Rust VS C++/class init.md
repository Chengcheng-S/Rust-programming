## Class Member Initialisation

C++不强制要求初始化每隔构造函数中的所有的变量

- C++ 11允许类具有默认成员初始化器，这些缺省成员初始化器在构造函数设置不存在时使用
- 没有显示构造函数的C++成员必须显示初始化
- 引用的元素必须显式初始化
- 基本类型(指针在内) 没必要进行初始化
- 元素没必要按照声明的顺序进行初始化

```c++
class Coords {
public:
    double x = 0.0;
    double y = 0.0;
    double z = 0.0;
    // x and y are set with the inputs, z is set to 0
    Coords(double x, double y) : x(x), y(y) {}
};
```

Rust 在这方面比较严格，必须初始化结构中的所有成员。强制初始化结构的成员可确保结构始终处于一致的可预测状态。如果设置了所有字段，则初始化顺序无关紧要。

```rust
struct Coord {
  pub x: f64,
  pub y: f64,
  pub z: f64,
}
impl Coord {
  pub fn new(x: f64, y:f64) {
    Coord { x: x, y: y, z: 0f64 }
  }
}
///...
let coord1 = Coord::new(100f64, 200f64);
```

一个结构体实现一个或者多个`From<>` trait：

```rust
struct Coord {
  pub x: f64,
  pub y: f64,
  pub z: f64,
}
//1
impl From<f64,f64,f64> for Coord{
	fn from(value:(f64,f64,f64))->Coord{
		Coord{x: value.0, y: value.1, z: value.2 }
	}
} 

//2
impl From<(f64, f64)> for Coord {
  fn from(value: (f64, f64)) -> Coord {
    Coord { x: value.0, y: value.1, z: 0.0 }
  }
}
```