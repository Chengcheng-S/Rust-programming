## Lambda

lamdba表達式是一個匿名函數，可以在調用本身的範圍内聲明和傳遞。

### C++

```c++
float values[10] ={ 9, 3, 2.1, 3, 4, -10, 2, 4, 6, 7 };

std::stort(values,values+10,[](float a,float b)){
    return a<b;
}
```

lamdba將被傳遞給`std::sort`隨後根據條件進行篩選。

C++中的lambda可以從環境中捕獲變量，有效捕获变量的lambda将成为闭包。

```c++
auto v1 = 30.;
auto v2=2.;

auto multiply =[v1,v2](){return v1*v2};
auto sum =[&v1,&v2](){return v1+v2};

cout<<multiply() <<end1;
cout <<sum()<<end1;

v1=99;
cout<<multiply()<<end1;
cout<<sum()<<end1;
```

捕获还可以通过在capture子句中指定`=`或通过引用指定默认捕获模式，然后为特定变量指定捕获行为。

```c++
// Capture by value
auto multiply = [=]() { return v1 * v2; };
// Capture by reference
auto sum = [&]() { return v1 + v2; };
```

注：C++中的lambda表达式存在的问题：如果捕获超出范围的变量的引用，将会导致程序崩溃。

### Closures in Rust

Rust中的闭包类似于lambda表达式，会自动在环境中捕获变量

```rust
use std::cmp::Ord;
let mut values = [ 9.0, 3.0, 2.1, 3.0, 4.0, -10.0, 2.0, 4.0, 6.0, 7.0 ];
values.sort_by(|a, b| a < b );
println!("values = {:?}", values);
```

更改捕获的值，需要使用关键字`move`,使其拥有这个变量的所有权，并且在外部范围内是不可访问的。

```rust
let mut x = 100;
let square = move || x * x;
println!("square = {}", square()); // 10000
x = 200;
println!("square = {}", square()); // 10000
```



















