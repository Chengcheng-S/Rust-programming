## Multhreading

多线程允许并发运行部分程序，并行执行任务。每个任务都有一个主线程`main()`的起始线程，此外还有自定义的主线程。

### 线程安全

- 线程不能同时修改数据，这种情况发生时，称为数据竞争，可能会损坏数据，导致崩溃。
- 线程不能以可能导致死锁的方式来锁定资源，即线程1或等资源b上的锁和资源a上的lock 而线程2获得资源a上的锁和资源B上的lock。两个线程都被永远锁定，等待一个永远不会释放的资源释放。
- 竞争，线程的执行顺序会对来自同一条输入的输出产生不可预测的结果
- 可由多个线程调用的api必须保护它们的数据结构，或者使其成为客户机要解决的显式问题。
- 必须安全地管理由多个线程访问的打开的文件和其他资源。

### 保护共享数据

不能在另一个线程同时读取数据，也不能两个线程同时写入数据

常见的方法：

- 使用互斥锁来保护对数据的访问：互斥锁是一个特殊的类，一次只能锁定一个线程，其他尝试锁定互斥锁的线程将等待另一个线程持有的锁被放弃。
- 读写锁，与互斥锁类似，允许一个线程锁定线程以写入数据，但是允许多个线程同时读取数据，前提是没有任何线程在写入数据。对于读取频率高于修改频率的数据，这比仅使用互斥锁要高效的多。

### 死锁

避免死锁的方法就是只获取一个对象的锁，并在完成后立即释放。但是必须要锁定多个对象，需要确保线程之间的锁定顺序是一直的，因此，如果线程1锁定A和B，那么确保线程2也按顺序锁定A和B，而不是锁定B和A。后者肯定会导致死锁。

### C/C++

常见的API：

- 从c++开始 `<thread>`  `<mutex> `
- POSIX线程或phreads,由POSIX系统公开，在win32线程之上构建了phread-win32支持
- W32线程
- OpenMP，由C++编译器支持
- Boost和Qt，第三方库提供了包装器来抽象线程api之间的差异

Api的共同的：

- 线程创建、销毁、连接和分离
- 使用锁和屏障在线程之间进行同步
- 互斥锁——  保护共享数据
- 条件变量——一种信号和通知条件为真的方法

#### std::thread

`std::thread` 表示单个执行线程，并提供了对依赖平台的线程方式的抽象

```c++
#include <iostream>
#include <thread>

using namespace std;

void DoWork(int loop_count){
    for(int i=0;i<loop_count;i++){
        cout<<"the number is "<<i<<endl;
    }
}
int main(){
    thread worker(DoWork,100);
    worker.join();
}
```

该示例生成一个线程，该线程调用函数并将参数传递给它，打印消息100次。

#### std::mutex

C++提供了一系列的互斥类型来保护对共享数据的访问。

互斥锁通过`lock_guard`，其他获取互斥锁的尝试将被阻止，直到释放锁为止。

```c++
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;

int result =0;
void DoWork(int loop_count) {
    for (auto i = 0; i < loop_count; ++i) {
        lock_guard<mutex> guard(data_guard);
        result += 1;
    }
}

int main(){
    thread worker1(DoWork,100);
    thread worker2(DoWork,50);
    worker1.join();
    worker2.join();
    cout<<"result="<<result<<endl;
}
```

#### POSIX threads

pthreads API以`pthread_`为前缀:

```c++
#include <iostream>
#include <pthread.h>

using namespace std;

void *DoWork(void *data) {
    const int loop_count = (int) data;
    for (int i = 0; i < loop_count; ++i) {
        cout << "Hello world " << i << endl;
    }
    pthread_exit(NULL);
}

int main() {
    pthread_t worker_thread;
    int result = pthread_create(&worker_thread, NULL, DoWork, (void *) 100);
    // Wait for the thread to end
    result = pthread_join(worker_thread, NULL);
}
```

#### Win32 thread

Win32线程具有类似于POSIX的功能  `CreateThread` `ExitThread` `SetThreadPriority`

#### OpenMP API

openmulti-Processing（OpenMP）是一个用于多线程并行处理的API。OpenMP依赖于编译器支持，因为您在源代码中使用`#pragma`指令来控制线程创建和数据访问。

OpenMP是一个复杂的标准，但是使用指令可以比直接调用线程api更干净地编写代码。缺点是它也更加不透明，隐藏了软件正在做的事情，使得调试更加困难。

#### Thread local storage

线程本地存储（TLS）是每个线程专用的静态或全局数据。每个线程都有自己的数据副本，因此可以修改它而不必担心引起数据争用。

编译器也有专有的方法将类型修饰为线程本地类型：

```c++
__thread int private; // gcc / clang
__declspec(thread) int private; // MSVC
```

C++ 11已经获得了一个TraceLead本地指令来装饰变量，这些变量应该使用TLS。

```c++
thread_local int private
```

### Rust

1. 必须以线程安全的方式保护所共享的数据
2. 在线程之间传递的任何数据都必须标记为线程安全。

#### Spawning the thrad

```rust
use std::thread;

thread::spawn(move||{
    println!("the thread");
});
```

```rust
fn my_thread(){
	println!("hello");
}
thread::spawn(my_thread);
```

如果提供一个闭包，那么它必须有一个`‘static`的生存期，因为线程可以比创建它们的线程活得长。i、 默认情况下，它们是分离的。

闭包可以使用标记为`Send`的`move`值，编译器允许在线程之间传输所有权。

类似的 闭包也可以返回一个标记为`Send`的值，这样编译器就可以在终止线程和调用`join`以获取该值的线程之间转移所有权。

所以上边的线程是分离的，若要等待线程完成，那么spawn会返回一个`joinHandle` 如此可以调用`join`来等待终止。

```rust
let h = thread::spawn(move ||{
    	println!("the rust thread");
});
h.join();
```

若闭包返回一个值，可以使用`join` 来捕获

```rust
let h= thread::spawn(move ||100*100);
let result =h.join().unwrap();
println!("result={}",result);
```

#### 编译器中的数据竞争保护

保护数据的最简单方法是将数据包装到互斥锁中，并为每个线程实例提供一个引用计数的互斥锁副本。

```rust
let share_data = Arc::new(Mutex::new(MyShareData::new()));

let share_data=share_data,clone();
thread::spawn(move||{
   let mut share_data=share_data.lock().unwrap();
    share_data.counter+=1;
});
```



```rust
struct MySharedData {
  pub counter: u32,
}
impl MySharedData {
  pub fn new() -> MySharedData {
    MySharedData {
      counter: 0
    }
  }
}
fn main() {
  spawn_threads();
}
fn spawn_threads() {
  let shared_data = Arc::new(Mutex::new(MySharedData::new()));
  // Spawn a number of threads and collect their join handles
  let handles: Vec<JoinHandle<_>> = (0..10).map(|_| {
    let shared_data = shared_data.clone();
    thread::spawn(move || {
      let mut shared_data = shared_data.lock().unwrap();
      shared_data.counter += 1;
    })
  }).collect();
  // Wait for each thread to complete
  for h in handles {
    h.join();
  }
  // Print the data
  let shared_data = shared_data.lock().unwrap();
  println!("Total = {}", shared_data.counter);
}
```

#### Read Write Lock

读写锁的工作原理类似于互斥锁，将共享数据包装在一个RwLock中，然后再包装一个Arc

```rust
let share_data= Arc::new(RWlock::new(MyShareData::new()));
```

之后每个线程都需要获得共享数据的读锁或写锁

```rust
let share_data= share_data.read().unwrap();

// or
let mut share_data = share_data.write().unwrap();
```

RwLock的优点是许多线程可以并发地读取数据，前提是没有任何内容写入数据。这在许多情况下可能更有效。

#### 第三方库 `Rayon`

Rayon 实现了并行迭代器，允许并行迭代集合。

```rust
use rayon::prelude::*;
fn sum_of_squares(input: &[i32]) -> i32 {
    input.par_iter()
         .map(|&i| i * i)
         .sum()
}
```



