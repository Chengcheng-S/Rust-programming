# any

std::any

这个模块实现trait，它可以在**运行时**动态的**输入任何类型的类型反射**

Any本身可用于获取TypeId，并在使用时具有更多的功能，作为特征对象。

作为&dyn Any（借用来的特征对象）它具有**is**和**downcast_ref**方法，以测试所**包含的值是否为给定类型，并且获取对内部值作为类型的引用**

作为&mut dyn Any 也是downcast_mut 方法，用于获取对内部数据的可变引用。

Box<dyn Any>添加了dowuncast方法，该方法尝试转为Box<T>

注： &dyn Any 仅限于测试值是否为指定的具体类型，不能用于测试类型是否实现特征

注销传递给函数的值，直到正在实现的值实现了Debug，但是不知道其具体类型， 对某些类型给予特殊待遇： 

```rust
use std::fmt::Debug;
use std::any::Any;

// log function 为any类型实现了debug
fn log<T:Any+Debug>(value:&T){
    let value_any=value as &dyn Any;
	//   将值转为String。如果成功，输出string的长度的值，如果失败则是为不同的类型 ，打印即可  
    match value_any.downcast_ref::<String>(){
        Some(as_string)=>{
            println!("String {}: {}",as_string.len(),as_string);
            
        }
        None=>{
            println!("{:?}",value);
        }
    }     
}

fn do_work<T>(value:&T)
	where T:Any+Debug
{
    log(value);
}
fn main(){
    let my_string="hello world".to_string();
    do_work(&my_string);
    let my_i8:i8=12;
    do_work(&my_i8);
}
```

## struct

TypeId  表示类型的全局唯一标识符

## traits

Any：  模拟动态类型的trait



## functions

type_name : 以字符切片的形式返回名字

type_name_of_val :以字符串切片形式返回指向值的类型的名称。类似于type_name::<T>(), 可用于变量类型不易获得的情况。
