## Functions

C++中的标准格式：

```c++
//Declaration
int foo(bool parameter1,const std::string &parameters);

//Implementation
int foo(bool parameter1,const std::string &parameters){
    return 1;
}
```

通常将函数声明为源文件中断额正向引用或头文件中的正向引用，然后再源文件中实现该函数。

若函数不反悔任何类型可以声明为`void`，若函数中有返回值，每个分支都应该有return语句的存在。

不需要声明函数的情况：

1. 若函数是内联的，即以内联关键字作为前缀，这个情况下，函数的实体是在一个地方声明和实现的。
2. 若函数不是内联函数，而是在同一源文件中调用它的程序之前声明的。

Rust中上述函数的等效声明为：

```rust
fn foo(parameter1:bool,parameter2:&str)->i32{
	//implementation
}
```

函数没有返回值的话将省略`->` 部分，函数也可以在其调用之后声明。若要在外部使用该方法需声明为`pub fn `

`return` 关键字是可以省略的，若要提前返回可以使用return。

### Variable arguments

C++中可以使用`...` 改变变量的数量，以此来实现可变参数

```c++
void printfs(const char*pattern,...);
```

Rust中并不支持可变参数

### Default arguments

C++中可以是设置参数的默认值

```c++
std::vector<Record> fetch_database_records(int number_to_fetch = 100);
```

### Function overloading

C++中的函数可以重载

```c++
std::string to_string(int x);
std::string to_string(float x);
```

Rust并不支持重载，函数的每个变体都必须有唯一的名称。

### c++11 替代语法

```c++
auto function_name(type parameter1, type parameter2, ...) -> return-type;
```

