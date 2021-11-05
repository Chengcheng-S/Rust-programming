## LOOP

### C++

#### For

C/C++中的for循环由包含在For()部分中的3个表达式部分和一个代码块执行：

- 零个或多个要初始化的变量(可以为空)
- 零个或多个条件为真，循环才能继续(可以为空)
- 在每次迭代中执行零个或多个操作(可以为空)

```c++
for(;;){
	//do something
}
```

#### range

C++循环由初始化表达式、条件表达式和由分号分隔的循环表达式组成。

```c+++
for(int i=0;i<100;i++){
	cout<<"Number"<<i<<endl;
}
```

#### interating

C++将迭代器的概念引入到集合类中。迭代器可以递增或递减以遍历集合。
因此，要将集合从一端迭代到另一端，迭代器将被分配集合的`begin()`迭代器并递增，直到它与`end()`迭代器匹配为止。

```c++
for (std::vector<string>::const_iterator i = my_list.begin(); i != my_list.end(); ++i ) {
  cout << "Value = " << *i << end;
}
```

C++ 11在迭代数组和集合时提供了新的基于循环的范围更简洁的语法：

```c++
std::vector values;
...
for (const auto & v: values) {
  ...
}
int x[5] = { 1, 2, 3, 4, 5 };
for (int y : x) {
  ...
}
```

#### Infinite Loop

无限循环 使用while或者 空的for循环

```c++
while(true){
    do_work();
}

for(;;){
    do_work();
}
```

#### While

C++有条件`while` 和`do while`形式，前者在表达式运行之前调用它，后者则是表达式之前最少执行一次。

```c++
while (!end) {
  std::string next = getLine();
  end = next == "END";
}
```

在C++中`do while`表单将至少执行一次循环体，因为该条件仅在每次迭代之后而不是在之前进行。

```c++
int i=0;
do {
	i=rand();
}while (i<10)
```

#### Break and Continue

如果需要退出循环或提前开始下一个迭代，那么可以使用`break`和`continue`关键字。break关键字终止循环continue，使循环进入下一个迭代。

```c++
bool foundAdministrator = false;
for (int i = 0; i < loginCredentials; ++i) {
   const LoginCredentials credentials = fetchLoginAt(i);
   if (credentials.disabled) {
     // This user login is disabled so skip it
     continue;
   }
   if (credentials .isAdmin) {
     // This user is an administrator so no need to search rest of list
     foundAdministrator = true;
     break;
   }
   // ...   
}
```

### Rust

#### For

Rust中`for`循环是迭代器的语法糖，若结构体实现了`interator`trait 那么可以使用`for` 对其进行迭代：

```rust
If structure type can be turned `IntoIterator`
  Loop
   If let Some(item) = iterator.next() {
     do_action_to_item(item)
   Else
     break;
  End
Else 
  Compile Error
Done
```

#### range

Rust中的range对象表示为`from` `to` 其中`from` `to` 是计算为值的值或表达式

```rust
let range=0..33;

let min =0;
let max =100;
let ranges = min..max;
```

Rust中的range，最大值是不包含在内的，最小值包含在内，

例如： 0到9

```rust
for i in 0..10{
	println!("{}",i);
}
```

0..10是从0到10的范围。范围实现`iterator` trait，因此for循环一次前进一个元素，直到到达末尾。

`enumerate()`函数。这会将迭代器转换为返回一个包含**索引和值的元组**，而不仅仅是值。

```rust
for (i,x) in (30..50).enumerate(){
	println!("index {} value {}",i,x);
}
```

#### For循环迭代数组和集合



1. ```rust
   let values = [1,4,6,7,8,9];
   for v in &values{
   	println!("{}",v);
   }
   ```
   1. 注：只能通过引用对数组进行迭代，通过值对数组进行迭代是不被允许的。

2. ```rust
   let values =[1,4,6,7,8,9];
   for v in values.iter(){
       println!("{}",v);
   }
   ```

3. 若集合是一个map类型，将返回`key` `value`

   ```rust
   use std::collections::HashMap;
   
   let mut values = HashMap::new();
   values.insert("rust","lang");
   for (k,v) in &values{
       println!("{},{}",k,v);
   }
   ```

4. 再有就是使用`for_each`

   ```rust
   let value= [1,4,6,7,8,9];
   value.iter().for_each(|x|println!("{}",x));
   ```

#### Break 和Continue

> Rust还有break和continue关键字，它们的操作方式类似于最内部的循环。continue将在下一次迭代中开始，而break将终止循环

```rust
let values = vec![2, 4, 6, 7, 8, 11, 33, 111];
for v in &values {
  if *v % 2 == 0 {
    continue;
  }
  if *v > 20 {
    break;
  }
  println!("v = {}", v);
}
```

#### Labels

某些情况下 可以通过设置labels的方式 当到达某个条件时进行`break`或`continue`

```rust
'x: for x in 0..10{
	'y: for y in 0..10{
		if x==5 && y==5{
            break 'x;
        }
        println!("x={},y={}",x,y);
	}
}
```

#### Infineite Loop

无限循环

```rust
loop{
	poll();
    do_work();
}
```

`break`可以中断无限循环

#### while loop

类C/C++式

```rust
while result <1024{
    process_request();
    result=result+1;
}
```

#### while let loop

```rust
let mut iterator =vec.into_iter();
while let Some(value)=iterator().next(){
    process(value);
}
```









