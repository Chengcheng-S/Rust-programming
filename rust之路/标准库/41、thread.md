# std::thread

本地线程

### The threading model

一个正在执行的Rust程序由一组**本机OS线程**组成，每个线程都有自己的堆栈和本地状态，线程可以命名，并为低级同步提供一些内置支持。

线程之间的通信可以通过通道、Rust的消息传递类型以及其他形式的线程同步和共享数据结构来完成，尤其是，保证线程安全的类型可以使用原子引用计数容器`Arc`在线程之间共享。

> Rust中致命的逻辑错误会导致线程死机，在此期间线程将展开堆栈，运行析构函数并释放所拥有的资源。
>
> Rust中的panic 可以通过`catch_unwind`恢复(使用编译参数 painc=abort)，或者使用`resume_unwind`恢复。
>
> 若没有捕捉到panic，线程将退出， 但可以选择从具有join的其他线程检测到死机。如果主线程在没有捕捉到panic的情况下死机，应用程序将使用非零退出代码退出。

**Rust 程序的主线程终止时，整个线程都会关闭，**即使其他线程仍在运行，这个模块提供了方便的工具来**自动等待子线程（即join）的终止**。

### Spawning a thread

生成一个新的线程 `thread::spawn`

```rust
use std::thread;
thread::spawn(move||{

});
```

父线程可以等待子线程的完成； 调用`spawn`下的`join`方法

```rust
use std::thread;

let child = thread::spawn(move || {
    // some work here
});
// some work here
let res = child.join();
```

join方法返回一个thread:：Result，其中包含子线程产生的最终值的Ok，或者返回给panic调用的值的Err！(如果child is panic)

### Configuring   threads

新线程可以通过`Builder` 类型派生之前进行配置，当前允许**配置子线程的名称和堆栈大小**。

```rust
use std::thread;
thread::Builder::new().name("child thread".to_string()).spawn(move||{
    	println!("the child thread");
})
```

### The  Thread type

线程通过Thread类型表示，可以用以下的方式获取：

- 通过生成新的线程： eg  使用`thread::spwan` 方法并且调用`JoinHandle`上的`thread`
- 请求当前线程， 通过使用`thread::current`方法

`thread::current`函数可用于不是由该模块的api派生的线程。

### Thread-local storage

本模块提供了一个Rust程序的**线程本地存储**的实现。线程本地存储是一种将数据存储到全局变量中的方法，**程序中的每个线程都有副本。线程不共享此数据，因此访问不需要同步**。

thread-local 键拥有它所包含的值，并在线程**退出时销毁该值**。它是用`thread_local!`创建的，并且可以包含任何静态值(没有借用的指针)。提供了一个访问函数`with`,生成对指定闭包的值的共享引用。

thread-local key 只允许对值进行共享访问，若允许可变借用，就不能保证**唯一性**。   大多数值都希望通过Cell或RefCell类型利用某种形式的内部可变性

### Naming threads

线程可以有关联的名称，便于识别。默认情况下，生成的线程是未命名的。要知道线程的名称，使用`Builder`生成线程，将名字传递给`Builder::name()`

从线程中检索线程名称，使用`thread::name`

- 若命名线程中出现painc，线程名称将打印在panic消息中
- 在适用的情况下，将线程名称提供给操作系统(例如，类unix平台中的pthread_setname_np)

### Stack size

派生线程默认堆栈大小为2MB, 这个堆栈大小可以进行更改，方式：

- 使用`Builder`创建一个线程并且，将需要的堆栈大小传递给`Builde::stack_size`
- 设置环境变量`RUST_MIN_STACK`  为一个表示堆栈的大小的整数(字节)，注：设置`Builde::stack_size` 将覆盖此值

注：**主线程的堆栈大小不是由Rust决定的**

### Type Definitions

Result				线程专用Result类型

### Structs

| AccessError | `LocalKey::try_with`返回的错误类型  |
| ----------- | ----------------------------------- |
| Builder     | Thread 工厂，可用于配置新线程的属性 |
| JoinHandle  | 在线程上加入拥有的权限(终止时阻塞)  |
| LocalKey    | 拥有其内容的线程本地存储密钥        |
| Thread      | 线程的句柄                          |
| Threadld    | 正在运行的线程的唯一标识符          |

### Functions

| current         | 获取调用它的线程的句柄                                       |
| --------------- | ------------------------------------------------------------ |
| panicking       | 确定当前线程是否因为panic而展开                              |
| park            | 块，除非或直到当前线程的 token可用                           |
| park_timeout    | 块，除非或直到当前线程的令牌可用或已达到指定的持续时间（可能错误地唤醒）。 |
| park_timeout_ms | 调用park_timeout(弃)                                         |
| sleep           | 当前线程睡眠时间                                             |
| sleep_ms        | 弃用                                                         |
| spawn           | 生成一个新线程，并且返回它的JoinHandle                       |
| yield_now       | 协调的将时间片交给操作系统调度器                             |
