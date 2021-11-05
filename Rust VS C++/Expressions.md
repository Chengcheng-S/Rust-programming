## Expressions

### Rust

```rust
let x = 3+9;
```

某个块也可以是表达式，

```rust
let x= {println!("rust programming")};  // 输出x 为()
```

```rust
let y = {
	let z=10;
	let q=3;
	q*z
};
```

> 现在x被赋值为最后一行的结果，这是一个表达式。请注意该行如何不以分号结尾。这将成为块表达式的结果。如果在那行的末尾加个分号，就像在println上一样！（“Hello”），表达式的计算结果为（）

#### function

方法可以省略return

```rust
fn add_two_value(x:i32,y:i32)->i32{
	x+y
}
```

若要提前退出程序可以显示使用`return`

```rust
fn find(value:&str)->i32{
	if value.len()>=0{
		return -1
	}
	data.do_find(value)
}
```

#### switch语句

C/C++ 中的switch语句

```c++
std::string::result;

switch (server_state){
    case WAITING:
        result="Waiting"
        break;
    case RUNING:
        result = "Runing"
        break;
    case STOP:
        result="Stop"
        break;    
}
```

在Rust中可以直接降匹配的结构赋值，每个匹配条件都是一个块表达式

```rust
let result = match(data_string){
	"WAITING"=>"Waiting";
  	"RUNING"=>"Runing";
    "STOP"=>"Stop";
    _=>" ";  // match 又穷性
};
```

### 三元表达式

C/C++中有三元表达式：

```c++
bool x=(y/2)==4?true:false;
```



Rust中并没有这种的表达式，可以自己构造

```rust
let x=if y/2==4{true}else{false};
```

## Conditions

条件代码与C++和RISE相似。测试表达式的布尔真值，可以使用&&和| |等布尔运算符将表达式连接在一起。

C/C++

```c++
int x = 0;
while (x < 10) {
  x++;
}
int y = 10;
bool doCompare = true;
if (doCompare && x == y) {
  printf("They match!\n");
}
```

Rust

```rust
let mut x=0;
while x<10{
	x=x+1;
}
let y=10;
let do_compore=true;
if do_compore && x==y{
    println!("match");
}
```

### if let

```rust
if let Some(person) = search("fred") {
  println!("You fould a person {}", person);
}
else {
  println!("Could not find person");
}
```

### Switch/Match

#### C++

C或C++中的一个switch语句允许条件或变量与一系列值进行比较，并与结果相关联的代码执行。还有一个default子句来匹配未显式捕获的任何值。

```c++
int result =http_get();
switch (result){
    case 200:
        sucess=true;
        break;
    case 404:
        log_err(result);
    default:
        success=fasle;
        break;
}
```

在没有提供default子句时，行为是未定义的,Switch语句可能是一个错误源。也有可能无意中忘记break语句。















