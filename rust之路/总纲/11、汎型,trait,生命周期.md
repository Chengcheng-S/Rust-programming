# 汎型

在Rust中，汎型是具體類型或其他屬性的抽象替代。在類理論中稱爲參數多態，用於表達抽象類型的機制，一般用於功能確定、數據類型待定的類

在聲明函數簽名或者結構體元素時使用汎型，隨後搭配不同的類型來使用這些元素

## 函數中

在函數體中使用時，在簽名中聲明對應的參數名稱，類似的，當在函數簽名中使用類型參數時，也需要在使用前生命這個類型，類型名稱必須被放置在函數名與參數列表之間的一對`<>` 中

```rust
fn  f1<T>(a:T)->T{
  // dosomething
}
```

## 用於結構體中

也可以使用`<>` 來定義結構體一個或多個字段

```rust
struct Person<T>{
	Name ：T,
	Hobby :T,
}
```

## 枚舉中

變體中存放汎型

```rust
enum Option<T>{
Some(T),
None
}
```

## 方法定義中

```rust
fn main() {
    let p = Person { x: 3, y: 4 };
    let x = p.getx();
    println!("{}", x);
}

struct Person<T> {
    x: T,
    y: T,
}

impl<T> Person<T> {
    fn getx(&self) -> &T {
        &self.x
    }
}
```

# trait

trait被用來向Rust編譯器描述某些特定類型擁有的且能被其他類型共享的功能，可以以一種抽象的方式來定義共享行爲，使用trait約束來將汎型參數指定為實現了某些特定行爲的類型

## 定義

類型的行爲由該類型本身可提供的方法組成，可以在不同的類型上調用相同的方法時，這些類型共享了相同的行爲

```rust
pub trait Name{
	fn f1(&self)->String;
}
```

關鍵字為`trait` 其後是名稱 f1是方法名，直接以`分號`結束當前的語句，任何想要實現這個trait的類型都需要為上述方法提供自定義的行爲

```rust
pub trait People{
    fn Show(&self)->String;
}

struct Person {
   pub x: i32,
    pub y: i32,
}

impl Person {
    fn getx(&self) -> &i32 {
        &self.x
    }
}

impl People for Person{
    fn Show(&self)->String{
        format!("p.x is {}, p.y is {}",self.x,self.y)
    }
}
```

類型實現trait和方法和類似，區別則是實現trait的時候需要在`impl` 后緊跟trait的名字 以及`for`關鍵字

### 孤儿原则

注：trait或其類型定義于庫中才可以使用，不能為外部類型實現外部trait，(孤兒規則：父類沒有定義在當前庫中，也是程序一致性的組成部分)

为某类型实现某 trait 的时候，必须要求类型或者 trait 至少有一个是在当前 crate 中定义的。你不能为第三方的类型实现第三方的 trait 。

- 如果要实现外部定义的trait需要先将其导入作用域
- 不允许对外部类型实现外部triat
- 可以对外部类型实现自定义的trait
- 可以对自定义的类型实现外部triat

## 默認實現

為trait中的某些方法提供默認行爲，當在實現某個特性類型trait時，可以保留或者重載每個方法的默認行爲

```rust
pub trait People {
    fn Show(&self) -> String {
        format!("this is name's people trait")
    }
    fn ShowSomething(&self) -> (&i32, &i32);
}

struct Person {
    pub x: i32,
    pub y: i32,
}

impl Person {
    fn getx(&self) -> &i32 {
        &self.x
    }
}

impl People for Person {
    fn ShowSomething(&self) -> (&i32, &i32) {
        (&self.x, &self.y)
    }
}

```

注：：無法再重載過程中調用該方法的默認實現

## trait作參數

使用trait定義接受不同類型參數的函數

```rust
fn f2(item:impl People){
	Print!("{}",item.people);
}
```

使用關鍵字`impl`以及對應的trait的名稱，這一參數可以接受任何類型的trait類型，也就可以調用任何來自People的方法

```rust
fn f2<T: People>(item:T){
    println!("{}", item.Show());
}
```

### 多個trait為參數

```rust
fn f3(item:impl A+B){// do somthing}

fn f4<T:A+B>(item:T){// do someting}
```

where的簡化

```RUST
fn f4<T>(item:T)
where T:A+B
{// do someting
}
```

### trait返回值

```rust
fn f4<T:A+B>(item:T)->impl A{// do someting}
```

### 使用trait約束有條件的實現方法

```rust
use std::fmt::Display;

struct Pair<T>{
      x:T,
      y:T,
}

impl<T>Pair<T>{
	fn new(x:T,y:T)->Self{
		Self{
		x,
		y,
		}
	}
}

impl<T:Display+PartialOrd>Pari<T>{
	fn cm_Display(&self){
		if self.x>=self.y{
			println!("the largest number is {}",self.x);
		}else{
		println!("the largest number is {}",self.y);
		}
	}
}
```

為實現了某個trait類型有條件的實現另一個trait，對滿足trait約束的所有類型實現trait成爲triat覆蓋

# 生命周期

Rust的每個引用都有自己的生命周期，他對應著引用保持有效性的作用域，絕大多數的時候，生命值周期都是被隱式的推導出來的。當生命周期可能以不同的方式相互關聯的時候，就需要手動的去標注生命周期，Rust需要開發者在標注汎型生命周期參數之間的關係，來確保運行時實際使用的引用一定有效。

## 懸垂引用

生命周期最主要的目的是避免懸垂引用，進而避免程序引用到非預期的數據

```rust
{
let r;   // r 進入作用域
{
 let x=5;    // x進入作用域
 r=&x;       //  借用
}             // x離開作用域
println!("{}",r);    
}// r離開作用域
```

```rust
error[E0597]: `x` does not live long enough
 --> src\main.rs:5:11
  |
5 |         r=&x;
  |           ^^ borrowed value does not live long enough
6 |     }
  |     - `x` dropped here while still borrowed
7 |     println!("r {}",r);
  |                     - borrow later used here

error: aborting due to previous error
```

上述錯誤的大致意思則是x的存活時間太短，r的作用域太大

### 借用檢查器

Rust編譯器擁有一個`借用檢查器`，用於比較不同作用域并確定所有借用的合法性。

## 生命周期

生命周期的標注并不會改變任何引用的生命周期的長度，(如同使用了汎型參數的函數可以接受任何類型一樣，使用了汎型生命周期的函數也可以接受帶有任何生命周期的引用)。在不影響生命周期的前提下，標注本身會被用於描述多個引用生命周期之間的關係。

#### 標注語法

其參數名稱必須以`'`開頭，且通常使用小寫字符。名稱通常非常簡短，將生命周期參數的標注填寫在`&`之後，并通過一個`&nbsp`來標注與引用類型區分開

```
&i32              // 引用
&'a i32           // 擁有顯示生命周期的引用
&'a mut i32       // 擁有顯示生命周期的可變引用
```

標注的存在是爲了向Rust描述多個汎型生命周期參數之間的關係。

假設一個函數的兩個參數X,Y 的同時擁有`&'a` 這就意味著：X,Y的引用必須與這裏的汎型生命周期存活時間一樣.

#### 函數簽名中的生命周期標注

**參數**和**返回值**中的所有引用都必須**擁有相同的生命周期**

```rust
fn f1<'a>(x:&'a str,y:&'a str)->&'a str{
	if x.len()>y.len(){
	x
	}else{
	y
	}
}
```

f1中的函數簽名表明,函數需要獲取兩個字符串切片參數的存活時間,必須不能小於給定的'a

當函數返回一個**引用**時,返回類型的生命周期參數必須要與其中**一個參數的生命周期參數相匹配**,如果返回的引用沒有指向任何的參數時,那麽它只能指向一個創建於函數内部的值,這個值也會在函數結束之後離開作用域,從而變成了懸垂引用

```rust
fn f2<'a>(x:&str,y:&str)->&'a str{
	let s=Stirng::from("hi");
	s.as_str()
}
```



```
rror[E0515]: cannot return value referencing local variable `s`
 --> src\main.rs:9:5
  |
9 |     s.as_str()
  |     -^^^^^^^^^
  |     |
  |     returns a value referencing data owned by the current function
  |     `s` is borrowed here
```

錯誤的原因是返回值的生命周期,沒有個任何參數的生命周期產生關聯.

從根本上來説,生命周期語法就是用來關聯一個函數中不同參數以及返回值的生命周期,一旦他們形成了某種聯係,Rust就會獲取足夠的信息來保障内存安全的操作,並阻止了那些可能會導致懸垂指針或者其他違反内存安全的行爲

#### 結構體中的生命周期

```rust
struct A<'a>{
    ONE: &'a str,
}

```

该引用必须比borrowed的生存周期长。

```rust
struct Borrowed<'a>(&'a i32);
```

#### 枚举

```rust
#[derive(Debug)]
enum Either<'a> {
    Num(i32),
    Ref(&'a i32),
}
```



### 生命周期省略

借用檢查器在這些某些情況下可以自動推斷出生命周期,而無需顯示的標注, 再次之前需要瞭解兩個概念

`輸入生命周期`:函數參數或者方法參數中的生命周期

`輸出生命周期`:返回值的生命周期

在未顯示標注的情況下,編譯器使用了三種規則來計算生命周期

- 每一個引用參數都會擁有自己的生命周期參數
- 當只存在一個輸入生命周期參數時,這個生命周期會被賦予所有的輸出生命周期參數
- 當擁有多個輸入生命周期時,其中一個是`&self`或者`&mut self`,self 的生命周期會被賦予所有的輸出生命周期參數

第一條規則適用於輸入生命周期,第二、三規則作用域輸出生命周期。當編譯器檢查完三條規則之後仍有無法計算出的生命周期，則會產生panic，這些規則不僅對fn有效，還對impl有效

### 靜態生命周期

`'static`表示在整個程序的執行期間，所有字符字面量都擁有`'static`生命周期，也可以被强制转换成一个更短的生命周期。

```rust
fn f1<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
where
    T: Display,
{
    println!("{}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

```

生命周期也是汎型的一種，所以'a和T可以被放置到<>中

有两种方式使变量拥有`‘static`生命周期，他们都把**数据保存在可执行文件的只读内存区**：

- 使用`‘static`声明来产生常量
- 产生一个拥有`&'static str`的`string`字面量

```rust
 {
        // 产生一个 `string` 字面量并打印它：
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);
        
    }
```

 当 `static_string` 离开作用域时，该引用不能再使用，不过
数据仍然存**在于二进制文件里面**。

### lazy_static

lazy_static在第一次调用时(运行时)进行初始化，而非编译时，常用的static则是在编译时进行初始化，运行阶段分配内存空间。

lazy_static是可以给静态变量延迟赋值的宏。

使用这个宏,所有 static类型的变量可在执行的代码在运行时被初始化。 这包括任何需要堆分配,如vector或hash map,以及任何非常量函数调用。

> 声明指令是为变量分配内存空间，赋值指令是把值保存到变量内存空间，声明指令在编译期间相对内存空间就已经被固定好，方法栈被创建时，就自动把变量和该内存空间联系起来。 分配内存空间，并初始化为0 声明语句在编译期间就已经为变量设定好内存相对地址了，即变量一上来就已经在栈的数据区分配好，即使代码指令没有执行，也不影响数据区的分配

一般情况下const类型函数只能被静态或常量表达式调用，

使用lazy_static可以使用hashmap声明常量

注： 使用之前需要在cargo.toml中添加依赖

```rust
[dependencies]
lazy_static="1.4.0"
```

每一个lazy_static **使用时必须要解引用**

```rust
#[macro_use]
extern crate lazy_static;

use std::collections::HashMap;

lazy_static! {
    static ref VEC:Vec<u8> = vec![0x18u8, 0x11u8];
    static ref MAP: HashMap<u32, String> = {
        let mut map = HashMap::new();
        map.insert(18, "hury".to_owned());
        map
    };
    static ref PAGE:u32 = mulit(18);
}

fn mulit(i: u32) -> u32 {
    i * 2
}

fn main() {
    println!("{:?}", *PAGE);
    println!("{:?}", *VEC);
    println!("{:?}", *MAP);
}
```

### 约束

- `T:‘a` ：在`T`中的所有引用都比生命周期`‘a`长
- T:trait+‘a   :   T 类型必须实现triat 并且在T中所有引用都必须比‘a存活更长。

### 强制转换

一个较长的生命周期，可以转换为较短的生命周期，使它在一个通常情况下不能工作的作用域内也能正常工作。强制转换可由编译器隐式地推导并执行，也可以通过**声明不同**的**生命周期**的形式实现。

```rust
fn main(){

    let first=78i32;
    {
        let second:i32=32;
        let rus=multiply(&first,&second);
        println!("first and second mul result is {}",rus);

        let f1=choose_first(&first,&second);
        println!("choice num is {}",f1);
    }

}

fn  multiply<'a>(first:&'a i32,second:&'a i32)->i32{
    first*second
}
fn choose_first<'a:'b,'b>(first:&'a i32,_:&'b i32)->&'b i32{
    first
}

```

本例中 first 拥有较长的生命周期，second拥有较短的生命周期，Rust推到了一个尽可能短的生命周期，然后两个引用被强制转换成了这个短的生命周期

**<‘a: ‘b ,‘b>读作生命周期a至少和b一样长。**

在choose_first函数中，强制转换&’a变为&’b





