# std::cell

可共享可变容器。

Rust memory safety基于以下规则：给定一个对象T，只能有以下一种：

- 对象有几个不可变的引用(也称为别名)
- 对象只能有一个可变的引用(可变)

`Cell<T>` 和`RefCell<T>` 允许以单线程方式执行此操作。二者都是非线程安全(均未实现Sync)，需要在多个线程间实现别名和可变时，可以使用`Mutex` `RWLock` `atomic types`

Cell 类型分为两种 Cell<T> 和RefCell<T>, `Cell<T>` 实现内部可变(通过将值移入和移除实现可变性)。使用引用 必须使用RefCell<T>类型。

检索和更改当前内部值得方法：

- 对于可`copy`得类型，`get` 方法检索当前得内部值
- 对于实现了`Default`得类型，`take`方法替换当前得内部值，使用`Default::default()`并返回替换后得值
- 对于所有得类型 `replace` 替换当前内部得值并返回替换后得值和`into_inner`方法消耗了`Cell<T>` 并返回内部值，  `set` 替换内部值，删除替换得值。

RefCell<T> 使用Rust的生命周期来实现“动态借用”，

```rust
  let shar_map: Rc<RefCell<_>> = Rc::new(RefCell::new(HashMap::new()));
    {
        let mut map = shar_map.borrow_mut();
        map.insert("gen", 0);
        map.insert("one", 1);
        map.insert("two", 2);
        map.insert("three", 3);
    }
    //
    let total:i32 =shar_map.borrow().values().sum();
    println!("{}",total);
```

Cell<T> 实现内部可变性
```rust
let a= A{Fileda:3,Filedb:Cell::new(32u32)};
    let newb:u32 = 78;
    a.Filedb.set(newb);
    println!("{:?}",a.Filedb);



struct A<T>{
    Fileda:u8,
    Filedb:Cell<T>
}
```
### swap
交换两个Cell中得值，不同于std::mem::swap  此处不需要`&mut`
```rust
use std::cell::Cell;

let c1 = Cell::new(5i32);
let c2 = Cell::new(10i32);
c1.swap(&c2);
assert_eq!(10, c1.get());
assert_eq!(5, c2.get());
```
>安全：如果从单独的线程中调用，这可能会有风险，但是`Cell`是`！Sync`，因此不会发生。这也不会使任何指针无效，因为`Cell`确保没有其他东西会指向这两个`Cell`中的任何一个

### replace
替换包含值，并且返回
```rust
    let a= A{Fileda:3,Filedb:Cell::new(32u32)};
    let newb:u32 = 78;
    // a.Filedb.set(newb);
    let cc=a.Filedb.replace(newb);
    println!("{:?},{}",a.Filedb,cc);
```

### into_inner()
解值
```rust
let cc = a.Filedb.into_inner();
println!("{}", cc);
```

### get
返回值得副本
```rust
 #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn get(&self) -> T {
        unsafe { *self.value.get() }
    }
```
在单线程中调用，可能会造成数据竞争。

### update
使用函数创建一个新值，并且返回新值
```rust
 #[inline]
    #[unstable(feature = "cell_update", issue = "50186")]
    pub fn update<F>(&self, f: F) -> T
    where
        F: FnOnce(T) -> T,
    {
        let old = self.get();
        let new = f(old);
        self.set(new);
        new
    }
}
```

### as_prt
返回Cell中基础数据得原始指针
```rust
pub const fn as_ptr(&self) -> *mut T {
        self.value.get()
    }
```

### get_mut
返回对数据得可变引用，此调用（在编译时）可变地借用了Cell，从而保证我们拥有唯一的引用。
```rust
 pub fn get_mut(&mut self) -> &mut T {
        self.value.get_mut()
}
```
### from_mut
在&mut T 中返回&Cell<T>
```rust
 let slice: &mut [i32] = &mut [1, 2, 3];
let cell_slice: &Cell<[i32]> = Cell::from_mut(slice);
let slice_cell: &[Cell<i32>] = cell_slice.as_slice_of_cells();
```

```rust
    pub fn from_mut(t: &mut T) -> &Cell<T> {
        // SAFETY: `&mut` ensures unique access.
        unsafe { &*(t as *mut T as *const Cell<T>) }
    }
}
```
### take
获取Cell的值，保留`Default::default`的位置
```rust
pub fn take(&self) -> T {
        self.replace(Default::default())
    }
}
```

```rust
let c = Cell::new(5);
let five = c.take();
assert_eq!(five, 5);
assert_eq!(c.into_inner(), 0);  // 重点于此
```

### as_slice_of_cell
将`Cell<[T]>` ===> `[Cell<T>]`
Cell<T>和 T有相同的内存布局

```rust
pub fn as_slice_of_cells(&self) -> &[Cell<T>] {
        // SAFETY: `Cell<T>` has the same memory layout as `T`.
        unsafe { &*(self as *const Cell<[T]> as *const [Cell<T>]) }
    }
}
```

```rust
let mut arr_a =[1,2,3,4]; // 此处需要设置变量
    let slice:&Cell<[_]>=Cell::from_mut(&mut arr_a); // 句末释放临时变量
    let cell_slcie = slice.as_slice_of_cells();
    println!("{:#?}",cell_slcie);
```

### set

设置包含的值

```rust
use std::cell::Cell;

letc c = Cell::new(5);
c.set(10);
```





