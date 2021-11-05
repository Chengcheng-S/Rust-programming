iter-itertrait

# Associate Types
type Item :  被迭代的元素类型

## 方法
### take_while
```rust

fn take_while<P>(self,predicate:P)->TakeWhile<Self,P>
where: P:FnMut(&self::Item)_>bool;
```
describe: 创建基于谓词生成元素的迭代器，
   `take_while`，以闭包作为参数。将对迭代器的每个元素调用此闭包，并返回true时生成元素
    在返回false之后，take_while的工作结束，其余元素全被忽略

```rust
fn main() {
   let a =[-1i32,0,1];
   let mut iter=a.iter().take_while(|x| x.is_negative());

   println!("{:?}",iter.next());
}

```
因为传递给take_while（）的闭包接受引用，而许多迭代器迭代引用，这可能会导致一种混乱的情况，其中闭包的类型是双重引用

```rust
fn main() {
   let a =[-1i32,0,1];
   let mut iter=a.iter().take_while(|x| **x<0);

   println!("{:?}",iter.next()); // Some(1)
   println!("{:?}",iter.next());  //  None
}

```
在初次遇到false时 停止，遇到fasle时take_while() 将不能使用
因为take_while()需要查看该值以确定是否应包含该值，因此使用迭代器将看到该值已被移除。

```rust
fn main() {
   let a =[-1,-3,0,3,5,7];
   let mut iter=a.iter();

   let result:Vec<i32>=iter.by_ref()
                        .take_while(|n| **n !=3 )
                        .cloned()
                        .collect();
   println!("get a value is {:?}",result);

}
```
get a value is [-1, -3, 0]

3已经不在那里了，因为它被使用是为了看看迭代是否应该停止，但是没有放回迭代器中。

### take
```rust
fn take(self, n: usize) -> Take<Self>
```
创建一个迭代器，返回其前n个元素

```RUST
fn main() {
   let a =[-1,-3,0,3,5,7];
   let mut iter=a.iter().take(3);
   println!("the iter's next element is {:?}",iter.next());
   println!("the iter's next element is {:?}",iter.next());
   println!("the iter's next element is {:?}",iter.next());
}
```
result:  
```rust
the iter's next element is Some(-1)
the iter's next element is Some(-3)
the iter's next element is Some(0)
```
常和无限迭代器一起使用，使得有限
```rust

let mut iter = (0..).take(3);

assert_eq!(iter.next(), Some(0));
assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), None);
```
如果可用的元素少于n个，take将把自身限制为底层迭代器的大小

```Rust

let v = vec![1, 2];
let mut iter = v.into_iter().take(5);
assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), None);
```

###  skip
```rust
fn skip(self, n: usize) -> Skip<Self>
```
创建一个迭代器，自动略过前n个元素

当它们被消耗后，剩下的元素就被释放了。与其直接重写此方法，不如重写第n个方法。

```rust
let a = [1, 2, 3];

let mut iter = a.iter().skip(2);

assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), None);
```
### next
```rust

fn next(&mut self)->Option<Self::Item>
```
推进迭代器并返回下一个值.

迭代结束时，返回None，单个迭代器实现可能会选择继续迭代，因此在此调用next可能返回某个项

###  size_hint
```rust
fn size_hint(&self)->(usize,Option<usize>)
```
返回迭代器剩余长度的边界。 返回一个元组，第一个元素是下限，第二个元素是上限。
返回的元组的后半部分是一个选项<usize>。此处的None表示没有已知的上限，或者上限大于usize。

迭代器实现不强制生成声明的元素数。一个有缺陷的迭代器可能产生小于元素下界或大于元素上限的结果。

主要用于**优化**

实现应该提供一个正确的估计，否则就违反了trait的协议。

对于任何的迭代器默认都实现返回(0,None)

```rust
fn main() {
    let a=[1,2,3];
    let iter=a.iter();
    println!("{:?}",iter.size_hint());
}
```
```rust
（3，Some(3))
```

```rust

fn main() {
   let iter=0..;
   assert_eq!((usize::MAX,None),iter.size_hint());
}
```
### count
```rust
fn count(self)->usize;
```
使用迭代器，计算迭代次数并返回。
此方法将重复调用next，直到没有元素，返回它看到的次数。
注意，即使迭代器没有任何元素，next也必须至少调用一次。

该方法**无法防止溢出**，因此使用超过usize:：MAX元素的迭代器元素计数会导致错误的结果或出现死机。
如果启用了调试断言，则会出现panic。

如果迭代器的元素超过usize:：MAX，则此函数可能会panic。
```rust
let a = [1, 2, 3];
assert_eq!(a.iter().count(), 3);

let a = [1, 2, 3, 4, 5];
assert_eq!(a.iter().count(), 5);
```
### last
```rust
fn last(self) -> Option<Self::Item>
```
消耗迭代器，返回迭代器的最后一个元素

此方法将计算迭代器，直到返回None。

在执行此操作时，它会跟踪当前元素。在None返回之后，last（）将返回它看到的最后一个元素。

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().last(), Some(&3));

let a = [1, 2, 3, 4, 5];
assert_eq!(a.iter().last(), Some(&5));
```
### nth
```rust
fn nth(&mut self, n: usize) -> Option<Self::Item>
```
返回迭代器中的第N个元素。

与大多数索引操作一样，计数从零开始，因此nth（0）返回第一个值，nth（1）返回第二个值，依此类推。

注：所有之前的元素以及返回的元素都将从迭代器中使用。即：之前的元素将被丢弃，并且在同一个迭代器上多次调用nth() 返回不同的值。

```rust
fn main() {
    let mut iter = [1, 2, 3, 3, 6].iter();
    println!("{:?}", iter.nth(0));

    println!("{:?}", iter.nth(0));

    println!("{:?}", iter.nth(0));

    println!("{:?}", iter.nth(0));
}
```
result:
```rust
Some(1)
Some(2)
Some(3)
Some(3)
```
如果n大于或等于迭代器的长度，nth（）将返回None。

### step_by

```rust
fn step_by(self,step:usize)->StepBy<Self>
```

从同一点开始创建一个迭代器，但在每次迭代中按给定的量递增。

注：

1. 不管给定的步骤如何，迭代器的**第一个元素都将始终返回**。

2. 在固定时间不被忽略的元素

   > StepBy的行为类似于sequence next（）、nth（step-1）、nth（step-1）…，但也可以自由地像sequence advance_n_and_return_first（step）、advance_n_和_return_first（step）…出于性能原因，某些迭代器可能会改变使用的方式。第二种方法会使迭代器提前，并且可能会消耗更多的项

```rust
fn advance_n_and_return_first<I>(iter: &mut I, total_step: usize) -> Option<I::Item>
where
    I: Iterator,
{
    let next = iter.next();
    if total_step > 1 {
        iter.nth(total_step-2);
    }
    next
}
```

注： 如果给定的步骤为0，则该方法将panic。

```rust
fn main() {
    let mut iter = [1, 2, 3, 3, 6]
                            .iter()
                            .step_by(3);
    println!("{:?}",iter.next());
    println!("{:?}",iter.next());
}
```
```rust
Some(1)
Some(3)
```

###  chain

```rust
fn chain<U> (self,other:U)->Chain<Self,<U as IntoIterator >::IntoIter>
  where  U:IntoIterator<Item=Self::Item>
```

获取两个迭代器，并在两个迭代器上依次创建一个新的迭代器
chain 将返回一个新的迭代器，该迭代器将首先迭代第一个迭代器的值，然后迭代第二个迭代器的值，
换言之：将两个迭代器链接在一起，形成一个链。

因为chain()的参数使用intointerator，所以可以传递任何可以转换为迭代器的内容，而不仅仅是迭代器本身。

```rust
fn main() {
   let a =[0,1];
   let b = [-1,-2,-3].iter();
   let mut iter=a.iter().chain(b);

    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
}
```

```
the iter's next element is Some(0)
the iter's next element is Some(1)
the iter's next element is Some(-1)
the iter's next element is Some(-2)
```

slices（&[T]）实现到迭代器中，因此可以传递给chain（）

```RUST
fn main() {
   let a =[0,1];
   let b = &[-1,-2,-3];
   let mut iter=a.iter().chain(b);

    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
}

```

### zip

```rust
fn zip<U>(self, other: U) -> Zip<Self, <U as IntoIterator>::IntoIter>
where
    U: IntoIterator,
```

zip 将两个迭代器压缩成一对迭代器,以元组的形式返回

zip()返回一个新的迭代器，该迭代器将迭代其他两个迭代器，返回一个元组，其中第一个元素来自第一个迭代器，第二个元素来自第二个迭代器。

换言之：将两个迭代器压缩成一个迭代器

如果任一迭代器返回None，则压缩迭代器的next将返回None。如果第一个迭代器返回None，则zip将短路，并且不会在第二个迭代器上调用next。

```rust
fn main() {
   let a =[0,1];
   let b = &[-1,-2,-3];
   let mut iter=a.iter().zip(b).step_by(1);

    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
}
```

result:

```
the iter's next element is Some((0, -1))
the iter's next element is Some((1, -2))
the iter's next element is None
the iter's next element is None
```

传参方面和chain很类似

可以压缩无限迭代器：

zip（）通常用于将无限迭代器压缩为有限迭代器。这是因为有限迭代器最终将返回None，从而结束zipper。使用（0..）压缩看起来很像枚举：

```rust
fn main() {
   let a =[0,1];
   let b = &[-1,-2,-3];
   let mut iter=(0..).zip(b);

    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
}

```

```rust
let enumerate: Vec<_> = "foo".chars().enumerate().collect();

let zipper: Vec<_> = (0..).zip("foo".chars()).collect();

assert_eq!((0, 'f'), enumerate[0]);
assert_eq!((0, 'f'), zipper[0]);

assert_eq!((1, 'o'), enumerate[1]);
assert_eq!((1, 'o'), zipper[1]);

assert_eq!((2, 'o'), enumerate[2]);
assert_eq!((2, 'o'), zipper[2]);
```

### map

```rust
fn map<B, F>(self, f: F) -> Map<Self, F>
where
    F: FnMut(Self::Item) -> B,
```

describe: 获取一个闭包并创建一个迭代器，该迭代器对每个元素调用该闭包。

map（）通过参数将一个迭代器转换成另一个迭代器：实现FnMut的东西。它生成一个新的迭代器，该迭代器对原始迭代器的每个元素调用此闭包。传递一个接受A并返回B的闭包。类似于for循环。map 惰型

```rust
fn main() {
   let a =[0,1];
   let b = [-1,-2,-3];
   let mut iter=b.iter().map(|x| -x );

    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
    println!("the iter's next element is {:?}",iter.next());
}
```

result

```
the iter's next element is Some(1)
the iter's next element is Some(2)
the iter's next element is Some(3)
the iter's next element is None
```

###  for_each

```rust
fn for_each<F>(self, f: F)
where
    F: FnMut(Self::Item),
```

对迭代器的某个元素调用闭包

这相当于在迭代器上使用for循环，尽管在闭包中不能使用break和continue。一般来说，使用for循环比较习惯，但是在较长的迭代器链的末尾处理项时，for每个循环都可能更易读。在某些情况下，for每个也可能比循环快，因为它将在诸如Chain这样的适配器上使用内部迭代。

```rust
use std::sync::mpsc::channel;

fn main() {
    let (tx,rx)=channel();
    (0..5).map(|X| X*2+1)
          .for_each(move |x| tx.send(x).unwrap());
    let mut v=rx.iter();
    println!("we receive from channel is {:?}",v.next());
    println!("we receive from channel is {:?}",v.next());
    println!("we receive from channel is {:?}",v.next());
    println!("we receive from channel is {:?}",v.next());
}
```

result:

```
we receive from channel is Some(1)
we receive from channel is Some(3)
we receive from channel is Some(5)
we receive from channel is Some(7)
```

###  filter

```rust
fn filter<P>(self, predicate: P) -> Filter<Self, P>
where
    P: FnMut(&Self::Item) -> bool,
```

返回一个迭代器，该迭代器使用闭包来确定是否应该产生一个元素

注：闭包必须返回true和false。 filter() 创建一个迭代器，对每个元素都调用这个闭包。若闭包返回true，则返回，否为false，将重试，并在下一个元素上调用闭包，看他是否能通过检测。

```rust
let a = [0i32, 1, 2];

let mut iter = a.iter().filter(|x| x.is_positive());

assert_eq!(iter.next(), Some(&1));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), None);

```

因为传递给filter()的闭包接受引用，而许多迭代器迭代引用，
这可能会导致一种混乱的情况，其中闭包的类型是双重引用

```rust
let a = [0, 1, 2];

let mut iter = a.iter().filter(|x| **x > 1); // need two *s!

assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), None);
```

通常在参数上使用析构来去掉一个

```rust
let a = [0, 1, 2];

let mut iter = a.iter().filter(|&x| *x > 1); // both & and *

assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), None);

```

或：

```rust
let a = [0, 1, 2];

let mut iter = a.iter().filter(|&&x| x > 1); // two &s

assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), None);
```

注： iter.filter(f).next() 相当于iter.find(f)



### filter_map

```rust
fn filter_map<B,P>(self,f:F)->FilterMap<Self,F>
where F:FnMut(Self::Item)->Option<B>
```

创建一个既可以map 又可以 filter的迭代器

filter_map 创建一个对每个元素调用闭包迭代器，切该闭包的返回值只能是Option<T>。

如果闭包返回Some(element),然后返回该元素。 若是返回None，将再次尝试并且调用这个闭包下的next方法，然后观察返回值是否为Some(element)

简言之：其会自动删除Option<T>层，若map已经返回了一个Option<T>,且希望跳过这个None，那么filter_map的效果更好

```rust
let a = ["1", "lol", "3", "NaN", "5"];

let mut iter = a.iter().filter_map(|s| s.parse().ok());

assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(3));
assert_eq!(iter.next(), Some(5));
assert_eq!(iter.next(), None);
```


### enumerate

```rust
fn enumerate(self)->Enumerate<Self>
```

创建一个当前迭代次数以及第一个值得迭代器，
结果返回：(i,val) 其中i是迭代的当前索引，val是迭代器返回的值。

enumerate() 让计数为usize。  zip函数允许使用不同大小的整数来计数

注：该方法无法防止溢出，因此枚举超过usize:：MAX元素的次数要么产生错误的结果，要么导致panic。如果启用了调试断言，则会出现panic。

```rust
fn main(){
    let a =["1","2","5"];
    let mut  iter=a.iter().enumerate();
    println!("{:?}",iter.next());

    println!("{:?}",iter.next());

    println!("{:?}",iter.next());

    println!("{:?}",iter.next());
}
```

result:

```
Some((0, "1"))
Some((1, "2"))
Some((2, "5"))
None
```

### peekable

```rust
fn peekable(self)->Peekable<Self>
```

创建一个使用peek来查看迭代器下一个元素，而不消耗迭代器。

注：第一次调用peek时，底层迭代器仍然是高级的，为了检查下一个元素，在底层迭代器上调用next，因此下一个方法的任何副作用(除了获取下一个值之外的任何东西)都将发生。

```rust
fn main(){
    let a =["1","2","5"];
    let mut  iter=a.iter().peekable();
    println!("{:?}",iter.peek());
    println!("{:?}",iter.next());


    println!("{:?}",iter.next());

    println!("{:?}",iter.next());

    println!("{:?}",iter.next());
}
```

###  skip_while

```rust
fn skip_while<P>(self, predicate: P) -> SkipWhile<Self, P>
where
    P: FnMut(&Self::Item) -> bool,
```

创建一个迭代器，跳过基于谓词的元素
接受闭包作为参数，它将对迭代器的每个元素调用此闭包，并忽略元素，直到返回false。
返回false之后，skip_while()的工作结束，剩下的元素也就返回。

```rust
fn main(){
    let a =[-1i32,0,1];
    let mut iter=a.iter().skip_while(|x| x.is_negative());
    println!("{:?}",iter.next());

    println!("{:?}",iter.next());

    println!("{:?}",iter.next());

    println!("{:?}",iter.next());
}
```

因为传递给skip_while()的闭包接受引用，并且许多迭代器迭代引用，这可能会导致一种混乱的情况，其中闭包的类型是双重引用：
遇到false则会停止。

###  map_while
```rust
fn map_while<B,P>(self,predicate:P)->MapWhile<Self,P>
```
创建一个迭代器，该迭代器基于predicate和map生成元素。
接收闭包作为参数，给每一个迭代器元素都调用该闭包，当结果为Some()返回元素
因为map_while（）需要查看该值以确定是否应包含该值，因此使用迭代器将看到该值已被移除


### scan
```rust
fn scan<St, B, F>(self, initial_state: St, f: F) -> Scan<Self, St, F>
where
    F: FnMut(&mut St, Self::Item) -> Option<B>
```
与fold类似的迭代器适配器，它保持内部状态并生成新的迭代器。
scan() 传递两个参数： 保持内部状态的整数初始值，以及一个闭包。 第一个是对内部状态的可变引用，第二个是迭代器元素，闭包可以分配给内部状态，移便在迭代之间共享状态。
迭代时，闭包将应用于迭代器的每个元素，并且闭包的返回值Option由迭代器生成。

```rust

fn main() {
    let a = [1, 2, 3];
    let  iter = a.iter()
        .scan(1, |st, &x| {
            *st = x+23 ;
            Some(*st)
        }
        );
    println!("{:?}", iter);
}
```
###  flatten
```rust
fn flatten(self) -> Flatten<Self>
where
    Self::Item: IntoIterator,
```
创建一个迭代器，展开其中的内容
当一个迭代器的迭代器或者一个可以转换为迭代器的事物的迭代器，并且删除一个间接级别时，这非常有用。 只能消除一个维度的数据。

###  fuse
```RUST
fn fuse(self)->Fuse<Self>
```
创建一个在返回第一个None结束的迭代器
```rust
fn main() {
    let a =[1,2,3];
    let mut iter=a.iter().fuse();
    println!("{:?}",iter.next());
    println!("{:?}",iter.next());
    println!("{:?}",iter.next());
    println!("{:?}",iter.next());
}
```
在迭代器返回None之后，将来的调用可能会再次产生Some(T).fuse（）调整迭代器，确保在给定None之后，它总是永远返回None。

### inspect
```rust
fn inspect<F>(self, f: F) -> Inspect<Self, F>
where
    F: FnMut(&Self::Item),
```
对每个元素进行迭代或传递。


```rust
let a = [1, 4, 2, 3];

// this iterator sequence is complex.
let sum = a.iter()
    .cloned()
    .filter(|x| x % 2 == 0)
    .fold(0, |sum, i| sum + i);

println!("{}", sum);

// let's add some inspect() calls to investigate what's happening
let sum = a.iter()
    .cloned()
    .inspect(|x| println!("about to filter: {}", x))
    .filter(|x| x % 2 == 0)
    .inspect(|x| println!("made it through filter: {}", x))
    .fold(0, |sum, i| sum + i);

println!("{}", sum);
```

### by_ref
```rust
fn by_ref(&mut self) -> &mut Self
```
借用迭代器，而不是使用它。
这有助于在保留原始迭代器所有权的同时应用迭代器适配器。


```rust
fn main() {
    let a =["3","None","1"];
    let mut iter=a.iter();
    let mut k =iter.by_ref().step_by(1);
    assert_eq!(k.next(),Some(&"3"));
}

```

###  collect
```rust
fn collect<B>(self) -> B
where
    B: FromIterator<Self::Item>,
```
将迭代器转为集合，常用于各种上下文。

使用collect()的最基本模式是将一个集合转换为另一个集合。获取一个集合，对其调用iter，执行一系列转换，然后在末尾collect()。

```rust
let a = [1, 2, 3];

let doubled: Vec<i32> = a.iter()
                         .map(|&x| x * 2)
                         .collect();

assert_eq!(vec![2, 4, 6], doubled);
```


```rust
let a = [1, 2, 3];

let doubled = a.iter().map(|x| x * 2).collect::<Vec<i32>>();

assert_eq!(vec![2, 4, 6], doubled);
```
collect并不对收集的数据类型感兴趣，因此可以使用`_`

```rust
let doubled = a.iter().map(|x| x * 2).collect::<Vec<_>>();
```

对于Result<T,E>，可以使用collect检查是否有任何结果失败
```rust
let results = [Ok(1), Err("nope"), Ok(3), Err("bad")];

let result: Result<Vec<_>, &str> = results.iter().cloned().collect();

// gives us the first error
assert_eq!(Err("nope"), result);

let results = [Ok(1), Ok(3)];

let result: Result<Vec<_>, &str> = results.iter().cloned().collect();

// gives us the list of answers
assert_eq!(Ok(vec![1, 3]), result);
```

### partition
```rust
fn partition<B, F>(self, f: F) -> (B, B)
where
    B: Default + Extend<Self::Item>,
    F: FnMut(&Self::Item) -> bool,
```
消耗一个迭代器，从中生成两个集合。
传递给partition（）的谓词可以返回true或false。partition（）返回一对，返回true的所有元素和返回false的所有元素。
```rust
fn main() {
    let a =[1,-1,0,2];
    let (t,f):(Vec<i32>,Vec<i32>)=a .iter().partition(|&n| *n-1>0);
    println!("{:?}",t);
    println!("{:?}",f);
}
```
result:
```
[2]
[1,-1,0]
```

### partition_in_place
```rust
fn partition_in_place<'a, T, P>(self, predicate: P) -> usize
where
    P: FnMut(&T) -> bool,
    Self: DoubleEndedIterator<Item = &'a mut T>,
    T: 'a,
```
根据给定的谓词对迭代器的元素重新排序，使所有返回true的元素在所有返回false的元素之前。返回找到的真元素数。
未维护已分区项的相对顺序。

```rust
#![feature(iter_partition_in_place)]

let mut a = [1, 2, 3, 4, 5, 6, 7];

// Partition in-place between evens and odds
let i = a.iter_mut().partition_in_place(|&n| n % 2 == 0);

assert_eq!(i, 3);
assert!(a[..i].iter().all(|&n| n % 2 == 0)); // evens
assert!(a[i..].iter().all(|&n| n % 2 == 1)); // odds
```

### is_partitioned
```rust
fn is_partitioned<P>(self, predicate: P) -> bool
where
    P: FnMut(Self::Item) -> bool,
```
检查此迭代器的元素是否根据给定的要求分取，以便于所有返回true的元素都在所有返回false的元素之前。

```rust

assert!("Iterator".chars().is_partitioned(char::is_uppercase));
assert!(!"IntoIterator".chars().is_partitioned(char::is_uppercase));
```

### cycle
```rust
fn cycle(self) -> Cycle<Self>
where
    Self: Clone,
```
无限重复迭代器
迭代器将从头开始，而不是在None处停止。再次迭代后，它将再次从开始处开始。
```rust
let a = [1, 2, 3];

let mut it = a.iter().cycle();

assert_eq!(it.next(), Some(&1));
assert_eq!(it.next(), Some(&2));
assert_eq!(it.next(), Some(&3));
assert_eq!(it.next(), Some(&1));
assert_eq!(it.next(), Some(&2));
assert_eq!(it.next(), Some(&3));
assert_eq!(it.next(), Some(&1));
```

### cmp
```rust
fn cmp<I>(self, other: I) -> Ordering
where
    I: IntoIterator<Item = Self::Item>,
    Self::Item: Ord,
```
从词典上比较这个迭代器的元素和另一个迭代器的元素。
```rust
use std::cmp::Ordering;

assert_eq!([1].iter().cmp([1].iter()), Ordering::Equal);
assert_eq!([1].iter().cmp([1, 2].iter()), Ordering::Less);
assert_eq!([1, 2].iter().cmp([1].iter()), Ordering::Greater);
```

### cloned
```rust
fn cloned<'a, T>(self) -> Cloned<Self>
where
    Self: Iterator<Item = &'a T>,
    T: 'a + Clone,
```
创建一个复制其所有元素的迭代器。
```rust
let a = [1, 2, 3];

let v_cloned: Vec<_> = a.iter().cloned().collect();

// cloned is the same as .map(|&x| x), for integers
let v_map: Vec<_> = a.iter().map(|&x| x).collect();

assert_eq!(v_cloned, vec![1, 2, 3]);
assert_eq!(v_map, vec![1, 2, 3]);
```
### copied
```rust
fn copied<'a, T>(self) -> Copied<Self>
where
    Self: Iterator<Item = &'a T>,
    T: 'a + Copy,
```
创建复制其所有元素的迭代器。

```rust
let a = [1, 2, 3];

let v_copied: Vec<_> = a.iter().copied().collect();

// copied is the same as .map(|&x| x)
let v_map: Vec<_> = a.iter().map(|&x| x).collect();

assert_eq!(v_copied, vec![1, 2, 3]);
assert_eq!(v_map, vec![1, 2, 3]);
```
### unzip
```rust
fn unzip<A, B, FromA, FromB>(self) -> (FromA, FromB)
where
    FromA: Default + Extend<A>,
    FromB: Default + Extend<B>,
    Self: Iterator<Item = (A, B)>,
```
将成对的迭代器，转为一对迭代器。unzip（）使用一个完整的pairs迭代器，生成两个集合：一个来自pairs的左侧元素，另一个来自右侧元素
相反于zip
```rust
let a = [(1, 2), (3, 4)];

let (left, right): (Vec<_>, Vec<_>) = a.iter().cloned().unzip();

assert_eq!(left, [1, 3]);
assert_eq!(right, [2, 4]);
```
### rev
```rust
fn rev(self) -> Rev<Self>
where
    Self: DoubleEndedIterator,
```
反转迭代器
通常，迭代器从左到右迭代。使用rev（）之后，迭代器将从右向左迭代。这只有在迭代器有结尾的情况下才有可能，因此rev（）只适用于doubleEnditerator。
```rust

let a = [1, 2, 3];

let mut iter = a.iter().rev();

assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&1));

assert_eq!(iter.next(), None);
```

### fold
```rust
fn fold<B, F>(self, init: B, f: F) -> B
where
    F: FnMut(B, Self::Item) -> B,
```
一种迭代器方法，它应用一个函数，产生一个单一的最终值。

fold（）接受两个参数：一个初始值和一个包含两个参数的闭包：“累加器”和一个元素。闭包返回累加器在下一次迭代中应该具有的值。

初始值是累加器在第一次调用时的值。

在将此闭包应用于迭代器的每个元素之后，fold（）返回累加器。

此操作有时称为“reduce”或“inject”。

当有一个东西的集合，并且想要从中产生一个值时，折叠是很有用的。

注意：

fold（）和遍历整个迭代器的类似方法对于无限迭代器可能不会终止，

即使对于结果在有限时间内可确定的特征也是如此。

尝试在组成迭代器的内部部分调用fold（）。
```rust
let a = [1, 2, 3];

// the sum of all of the elements of the array
let sum = a.iter().fold(0, |acc, x| acc + x);

assert_eq!(sum, 6);
```