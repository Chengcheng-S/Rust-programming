## Memory Alloction

创建驻留在堆上而不是堆栈上的对象，以及创建和销毁这些对象的方式。

### C++

C/C++的分配方式不同：

- `malloc`/`calloc`/`realloc()` `free()`
- `new()`  `delete()`  (仅限于c++)
- `new[]` `delete[]` 仅限于c++ 数组

`malloc()`  /`free()`  不会调用响应的构造函数或析构函数

`realloc()`  :分配一个新的内存，在释放原来的内存之前，复制现有内存的内容。

```c++
char *buffer = (char*) malloc(1024);

free(buffer);

Stack *stack =new Stack();

delete stack;

Node *nodes =new Node[100];

delete []nodes;
```

分配内存必须和释放内存相结合，如果忘记释放内存：

- 所有权可能会变得混乱，
- 未使用正确的new&delete，导致内存泄漏。
- 完全忘记释放内存导致内存泄漏
- 多次释放内存
- 调用悬垂指针，即指向已释放内存的指针
- 导致堆碎片的方式分配/释放。

C++有专门的智能指针，管理对象的生命周期，

```c++
{
  std::auto_ptr<Database> db(new Database());
  //... object is deleted when db goes out of scope
}
// C++11
{
  std::unique_ptr<Database> db(new Database());
  //... object is deleted when db goes out of scope
  std::unique_ptr<Node[]> nodes<new Node[100]);
  //... arrays of objects are supported too
}
// C++11
{
  std::shared_ptr<Database> db(new Database());
  // Reference count db
  setDatabase(db);
  //... object is deleted when last shared_ptr reference to it goes out of scope
  std::shared_ptr<Node[]> nodes<new Node[100]);
  //... arrays of objects are supported too
}
```

其他分配内存的方式：

> 实际上，每个C和C++库都有解决内存管理的方法。他们都有自己独特的所有权概念，每个概念通常都是不同的。Boost和Qt有自己的内存管理“智能”指针。Qt甚至要求创建对象的线程上的消息处理循环“稍后”删除某些对象。有些库甚至采用类似COM的模型，用智能指针对对象进行引用计数。大多数C库将公开一个alloc和free函数，用于创建和销毁调用方传递给API的上下文对象。
>
> 在某些情况下，内存分配甚至可以被覆盖和替换。在C语言中，标准malloc/free可以替代另一个内存分配器，

### Rust

Rust的分配比C/C++更加严格，对象的生命周期由编译器跟踪和强制执行，其中包括内存分配的对象。

在正常的安全编程中，没有显式的new/delete，因此无法忘记释放对象。没有指针，因此代码不能调用悬空指针或无意中调用空指针。

- `Box` 保存堆分配对象的指针，一个Box不能被clone，任何时候智能有一个所有者
- `Cell` 可变内存位置，可以保存任何类型的可复制类型，且其中的值是可以修改的
- `RefCell` 保存引用的可变内存位置

正确地定义了一个对象的生命周期，它就会正确地存在和消失。在许多情况下，这种寿命管理带有零运行时成本，或者如果存在成本，它只不过是用C/C++中正确编写的相同代码。

Rust要求大多数堆分配的内存包含在下面的一个或多个结构中。结构管理生命周期和对内部对象的访问，确保生命周期得到正确管理。

#### Box

```rust
struct Blob {
  data: Box<[u8; 16384]>
}
impl Blob {
  pub fn new() {
    Efficient {
      data: Box::new([0u8; 16384])
    }
  }
}
```

#### Cell

`Cell`可以用`get()`或`ste()`复制以覆盖自己的副本。由于内容必须是可复制的，它们必须实现Copy。

运行时成本为零，因此他不必跟踪借用，但限制是它只对副本类型有效。因此，它不适用于大型对象或深度复制对象。

#### RefCell

RefCell保存对mut或不可变地借用的对象的引用。这些引用是读写锁定的，因此有运行时开销，因为借用必须检查是否有其他对象已经借用了引用。

通常，一段代码可能会借用某个范围的引用，然后当它超出范围时，借用就会消失，重复借用就会panic

#### 引用计数

Rust实现Rc<>和Arc<>是为了对需要由代码的不同部分共享和使用的对象进行引用计数。Rc<>是单线程引用计数包装，而Arc<>是原子引用计数包装。根据线程是否共享对象，可以使用其中一个。

引用计数对象通常包含在`Box`、`Cell`或`Refcell`之中。因此多个结构可以保存对同一对象的引用。

#### RC

引用计数对象一次可以由多个所有者持有。每个own保存一个克隆的Rc<T>，但T内容是共享的。对对象的最后一次引用导致内容被销毁。



#### Arc

一个原子引用计数的对象，它的工作方式类似于Rc<T>，只是它使用了一个原子递增的计数器，这使得它是线程安全的。维护原子引用计数需要更多的开销。如果多个线程访问同一对象，使用Arc<T>。