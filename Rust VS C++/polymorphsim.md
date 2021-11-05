## polymorphism

### C++

C++有四种多态：

1. 函数名重载，使用不同参数的同一个函数的多个定义。
2. Coercion——隐式类型转换(将double赋值为int或bool)
3. Parametric——模板中参数的编译类型替换
4. inclusion ——使用虚拟方法对类及逆行子类型话会重载其功能

```c++
class Variant {
public:
  void set(); // Null variant
  void set(bool value);
  void set(int value);
  void set(float value);
  void set(Array *value);
};
```

> C++具有默认参数值和默认构造函数,可以使用签名调用一个函数，并在编译器解析后调用完全不同的参数，从而避免在不同情境下调用不同功能的同名函数。

### Rust

Rust对多态的支持：

1. 函数不支持重载
2. 使用`as`关键字在数字类型之间进行转换
3. 类似于C++的泛型
4. 不可继承

#### `From<>` 和`Into<>` trait

- `From<>` 将某个类型转换为另一种类型
- `Into<>` （在过程中使用） 进行类型转换



