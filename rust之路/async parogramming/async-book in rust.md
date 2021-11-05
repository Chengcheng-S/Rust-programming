# asyncchronous programming in Rust

項目地址： [Why Async? - Asynchronous Programming in Rust (rust-lang.github.io)](https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html)

源碼地址：[rust-lang/async-book: Asynchronous Programming in Rust (github.com)](https://github.com/rust-lang/async-book)

> 異步編程，又名異步，被廣大編程語言所支持的并發編程模型，運行開發者在少量OS綫程上運行大量的并發任務，同時通過`async\await`語法保留普通同步編程大部分的代碼風格。

### 異步與其他的編程模型

- `OS Thread` 不許對編程模型進行任何更改，是的表達并發性變得非常容易。但是綫程之間的同步變得很困難，并且性能開銷很大。綫程池雖然可以減輕部分負荷，但是不足以支持大規模IO密集型工作負載。
- `event-driven programming` 與回調結合，十分高效，但是會導致冗長的非綫性控制流、數據流和錯誤傳播通常難以跟蹤。
- `Coroutines` 和thread一樣，不需要更改編程模型，這使得他們易於使用，和async一樣，他們可以支持大量任務。但是他們抽象了對系統編程和自定義運行時實現者都和重要的低級細節。
- `The actor model` 將所有并發計算劃分爲actor 單元，這些單元通過易出錯的消息傳遞進行通信，就像在分布式系统中一样。 Actor 模型可以有效地实现，但它留下了许多实际问题未解决，例如流量控制和重试逻辑。

目前來説， `async` 適合Rust等低級語言的高性能實現，同時提供綫程和協程。

### Rust中的異步和其他語言

- `Future` 在Ruts中是惰性的，只有在輪詢(poll)是才會工作，放棄future會使得其挂起
- `async`在rust中是零開銷，意味著開發者只需要為使用的内容承擔成本即可，可以在沒有堆分配和動態調度的情況下使用async，還允許在嵌入式中使用async。
- Rust 沒有提供内置運行時，運行時則是由社區維護的crate提供
- Rust中提供了單綫程和多綫程運行時，具有不同的優缺點。

### async vs threads in rust

rust中異步主要代替方案是使用OS thread，直接通過`std::thread`或間接通過綫程池。從綫程到異步通常需要在實現和任何公開的公共接口進行大規模的重構工作。

- `OS Thread` 適用於少量的任務，綫程會帶來CPU和内存開銷。綫程之間的生成和切換十分消耗資源，即便是空的綫程也會消耗系統資源。綫程池可以幫助減輕其中一些的成本。但是綫程允許開發者重用現有的同步代碼，而無需對代碼進行大規模的修改。
- `Async` 顯著降低了CPU和内存開銷，尤其是有大量IO綁定任務的工作負載。異步是用少量的綫程來處理大量的任務。但是一部函數生成的狀態機以及每個可執行文件都綁定了一個異步運行時，異步運行Rust會導致更大的二進制文件。

若不是處於性能考慮，盡量選擇多綫程。



### Rust中的自定義并發模型

Rust 并不會强制開發者在async 和os thread中選擇一個，相反在同一個工程中可以同時使用二者，或者使用其他的編程模型，如 event-driven

## 異步Rust的狀態

異步Rust目前處於正在完善階段，因此有許多的特性將會被支持：

- 典型并發工作負載的出色的運行時性能
- 頻繁地與高級語言功能交互，如 生命周期和pin
- 同步和異步代碼之間以及不同異步運行時之間的一些兼容性約束
- 由於異步運行時和語言支持的發展，更高的維護負擔

async Rust 更難使用，并且會導致比同步 Rust 更高的維護負擔，作爲回報，可以為開發者提供一流的性能。async Rust 的所有部分都在不斷的完善中，其這些影響也會隨著時間的推移，越來越小。

### 語言和庫支持

儘管Rust本身支持異步編程，但是大多數的異步程序依賴於社區crates提供的功能。

- 標準庫提供了最基本的trait、types、function：如 `Future` trait
- Rust 編譯器直接支持`async/await`語法
- `futures` crate 提供了很多的type、func  macros，可以直接在任何支持異步的rust程序中使用
- 異步代碼的執行、IO和任務生成由**異步運行時**提供，如 Tokio、async-std  等。大多數異步應用程序和一些異步crate依賴于特定的運行時。

### 編譯和調試

async rust中的編譯錯誤滿足sync rust相同的最高標準，但是由於async rust通常依賴於更複雜的語言特性，如 生命周期，pin。

### 運行時錯誤

每當編譯器遇到異步函數時，都會在後臺生成一個**狀態機**。async rust 中的堆棧跟蹤通常來自這些狀態機的詳細信息。以及來自運行時的函數調用，因此解釋堆棧跟蹤可能比sync rust 更加複雜。

### 故障模式

在異步Rust中，很少有新的故障模式是可能的。如，在異步上下文中調用阻塞函數，或者錯誤地實現Future trait，此類錯誤可以通過編譯器檢查、甚至是單元測試

### 兼容性注意事項

異步和同步代碼不能總是自由組合， 如：不能直接從同步代碼中調用異步函數。同步代碼和異步代碼也有不同的設計模式。

即使異步代碼不能總是自由組合，一些crate依賴於特定的一部運行時才能實現，若是如此，需要在crate的依賴項中指定。

### 性能特點

asnyc rust 的性能取決於開發者使用的**異步運行時的**實現。大多數異步生態都假設一個多綫程運行時。這使得難以達到單綫程一部應用程序的理論性優勢，

另一個則是延遲敏感的任務，這對驅動程序、GUI應用程序等很重要，此類任務取決於運行時/操作系統的支持。



## async/.await 入門

`async/.await` 是Rust的内置工具，用於編寫異步函數。  `async`將代碼塊轉換成為實現`Future`trait的狀態機。雖然在同步方法中調用阻塞函數會阻塞整個綫程，但是阻塞的`Future` 將放棄對綫程的控制，允許其他的`Future` 運行。

于`Cargo.toml`中添加依賴

```cargo
[dependencies]
futures="0.3.4"
```

創建異步函數的方法：

```rust
async fn do_something(){}
```

`async fn`返回值為一個`Future` 對於任何的事情，`Future` 都需要在`executor`上運行。

```rust
use futures::executor::block_on;
async fn hello_world(){
    println!("hello world");
}

fn main(){
    let future=hello_world(); //onthing is printed
    block_on(future);    // future is run and print `hello world`
}
```

> `block_on` 阻塞當前綫程，知道提供的`future`運行完成，其他執行程序提供更爲複雜的行爲，如：多個future 運行在一個綫程上。



在`async fn`中可以使用`.await`等待另一個實現`Future` triat的完成， 不同於`block_on` `.await` 不會阻塞當前的綫程，而是異步等待`Future`完成，若`Future`被阻塞，則允許其他的`Future` 任務運行。

```rust
use futures::executor::block_on;
use std::future::Future;

fn main() {
    println!("Hello, world!");

    let f2 = third();
    block_on(f2);
}

async fn first() -> usize {
    println!("the first function");
    0
}

async fn second() {
    let x = first().await;  //此處使用的是await 等待 而不是block_on 阻塞綫程
    println!("{:?}", x);
    println!("tne second function");
}

async fn third() {
    let sec =second();
    futures::join!(sec);

    println!("the third function");
}
```

> `Join` 類似於`.await`但是可以同時等待多個future,若sec被阻塞，third 則會阻塞并讓給主綫程。



## 執行`Futures` 和`Task` 的原理

### `Future` trait

`Future` trait 是Rust異步編程的核心。`Future`是一種可以產生值的異步計算(即使值爲空，例如`()`)。

```rust
triat SimpleFuture{
    type Output;
    fn poll(&mut self,wake:fn())->Poll<Self::Output>;
}

enum Poll<T>{
    Ready(T),
    Pending,
}
```

可以通過調用`poll` 函數來讓`Future` 繼續工作，若`Future`完成，則返回`Poll::Ready(result)` ,若未完成則返回`Poll::Pending` 並安排在`Future`準備執行是調用`wake()` 函數。當`wake()` 被調用時，驅動`Future`的`executor` 會再次調用`poll`,如此`future` 才可以繼續執行。

若沒有`wake()` 執行器將無法知道特定的`future`何時繼續執行，並必須不斷輪詢每個`future`使用`wake()`,`executor`確切地知道那些`future`準備好被輪詢。

`Future` 模型允許將多個異步操作組合在一起，而無需中間分配。一次運行多個`future` 或將多個`future` 鏈接在一起可以通過無分配狀態機來實現：

```rust
//一个 SimpleFuture，它同时运行另外两个 future 以完成。  并发是通过每个 future 调用 `poll` 来实现的

pub struct join<FutureA,FutureB>{
    
    /*
    每個字段可能包含一個應該運行完成的future，如果future已經完成，則將該字段設置為`Nonce`,如此防止在完成后輪詢`future`
    */
    a:Option<FutureA>,
    b:Option<FutureB>,
}

impl<FutureA,FutureB>SimpleFuture for join<FutureA,FutureB>
where
    FutureA: SimpleFuture<Output = ()>,
    FutureB: SimpleFuture<Output = ()>,
{
    type Output = ();
    fn poll(&mut self,wake:fn())->Poll<Self::Output>{
        //完成future a
        if let Some(a) =&mut self.a{
            if let Poll:::Ready(()) = a.poll(wake){
                sele.a.take();
            } 
        }
        
        // 完成future b
         if let Some(b) = &mut self.b {
            if let Poll::Ready(()) = b.poll(wake) {
                self.b.take();
            }
        }
        
         if self.a.is_none() && self.b.is_none() {
            //都已經完成則退出
            Poll::Ready(())
        } else {
            // 一個或者兩個都返回`poll::Pending` 且仍然有要做的工作，黨可以執行時，將調用`wake()`
            Poll::Pending
        }
        
    }
}
```

多個連續的`future` 可以一個接一個的運行

```rust
//future 一個接一個運行，知道全部運行完畢
pub struct AndThenFut<FutureA, FutureB> {
    first: Option<FutureA>,
    second: FutureB,
}


impl<FutureA, FutureB> SimpleFuture for AndThenFut<FutureA, FutureB>
where
    FutureA: SimpleFuture<Output = ()>,
    FutureB: SimpleFuture<Output = ()>,
{
    type Output = ();
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if let Some(first) = &mut self.first {
            match first.poll(wake) {
               //如此已經完成了第一個future，之後進行第二個
                Poll::Ready(()) => self.first.take(),
                
                // 若運行到此，説明第一個future遇到了問題
                Poll::Pending => return Poll::Pending,
            };
        }
        //目前第一個future已經運行完成，可以去運行第二個
        self.second.poll(wake)
    }
}
```



真正的`Future`

```rust
trait Future {
    type Output;
    fn poll(
        //&mut Self ——> Pin<&mut Self>
        self: Pin<&mut Self>,
        //  wake: fn() ——> cx: &mut Context<'_>
        cx: &mut Context<'_>,
    ) -> Poll<Self::Output>;
}

```



##  `Wake` 喚醒任務

`future` 在第一次被輪詢是無法完成屬於正常現象，future確保一旦準備繼續工作時，就會再次輪詢，這種機制是通過`waker`完成的。

每次輪詢`future`時,他都會作爲`task`的一部分進行輪詢。**task是提交給`executor`的頂級`futures`**。

`Waker` 提供了一個`wake()` 的方法，來告訴executor相關的task應該被喚醒。當調用`wake()`時，executor 知道與wake相關聯的任務已經準備好繼續工作，並且應該再次輪詢future。

`waker`還實現了`clone()` 便於對其進行複製和存儲。

> 每次輪詢future時，都必須更新 Waker，因爲future可能已經使用不同的 Waker 轉移到不同的任務。當future在被輪詢后再task之間傳遞時，就會發生這種情況。



## `Executor`

Future executors 獲取一組頂級的 Futures 並在 Future 可以執行时通过调用 poll 將它們運行到完成。通常，执行程序将輪詢一次future以开始。当 Futures 通过调用 wake() 表明其準備好繼續執行時，他們會被放到消息隊列中並再次調用 poll，重複直到 Future 完成。



## `async/.await`

`async\.await`可以讓出當前綫程的控制而不是阻塞，允許其他代碼在等待操作完成時繼續執行。

使用`async`有兩種主要的方式：`async fn` 和`async block`,每個都返回一個實現`Future` trait的值

```rust
async fn foo()->u8{1}

fn bar()->impl Future<Output =u8>{
    
    async {
        let x:u8 = foo.await;
        x+5
    }
}
```

`async` 和其他的futrue 都是惰性的，他們在運行之前什麽都不回去做，運行Future的幾種方式是：`.await`,當在future上調用`.await`時，他將嘗試運行直到完成。如果`future`被阻塞，他將放棄對當前綫程的控制。若可以繼續執行時`Future` 將被executor 拾取並回復運行。從而運行`.await`解析。

#### `async lifetime`

不同於傳統函數，引用或其他非靜態參數的`async  fn` 返回一個受參數生命周期限制的`Future`

```rust
fn foo_expanded<'a>(x: &'a u8) -> impl Future<Output = u8> + 'a {
    async move { *x }
}
```

`async fn` 返回的future必須是`.await`而非靜態參數。在調用函數后立即等待future。

將帶有引用的承諾書作爲`async fn` 轉換爲靜態future 的一種常見方式是將參數與對async 塊内的`async fn` 的調用綁定在一起

```rust
fn bad() -> impl Future<Output = u8> {
    let x = 5;
    borrow_x(&x) // ERROR: `x` does not live long enough
}

fn good() -> impl Future<Output = u8> {
    async {
        let x = 5;
        borrow_x(&x).await
    }
}

```

**通過將參數移動到async塊中，延長了他的生命周期**以匹配從調用返回的Future的生命周期。

### `async move`

async 塊和閉包允許使用`move` 關鍵字，和普通閉包一樣，async move將擁有它引用變量的所有權，允許其在適當的範圍内存活，但放棄與其他代碼共享這個變量的能力。

> 多个不同的 `async` 块可以访问同一个局部变量 只要它们在变量的作用域内执行

```rust
async fn blocks() {
    let my_string = "foo".to_string();

    let future_one = async {
        // ...
        println!("{}", my_string);
    };

    let future_two = async {
        // ...
        println!("{}", my_string);
    };
    // 運行兩個future 直到完成
    let ((), ()) = futures::join!(future_one, future_two);
}
```



### `.await` 多綫程執行器

**使用多綫程future executor時，Future 可能會在綫程之間移動**，因此async block 使用的任何變量都必須是綫程之間可以移動的。(任何.await都可能導致切換到新的程序)

意味著 Rc、&RefCell或任何其他未實現Send trait的類型都是不安全的，包括對未實現`Sync` trait 的類型的引用

> 只有在調用 .await 期間不在範圍内，就可以使用這些類型

同理，避免產生死鎖，得使用`future::lock`中的互斥鎖，而不是`std::sync`中的互斥鎖。

## `Pin`

要輪詢`future`，必須使用`Pin<T>`的特殊類型固定他們，

`Pin` 與`Unpin` 協同工作，pin可以保證實現`!Unpin` 的對象永遠不會被移動。

`Pin<T>`類型包裝指針類型，保證指針後邊的值不會被移動，例如`Pin<&mut T>` `Pin<& T>` `Pin<Box<T>>` 都保證即使`T:!Unpin` 也不會移動`T`

> 大多數類型在移動時沒有問題，這些類型實現了`Unpin` trait,指向`Unpin`類型的指針可以自由的放入或去除`Pin ` 如：u8 是 Unpin，所以 Pin<&mut u8> 的行为就像一个普通的 &mut u8。

到那時，Pin后無法移動的類型有一個叫做`!Unpin`的例子，`async/.await`創建的`Future`就是一個例子。

將一個`!Unpin`類型固定到堆上為數據提供了一個穩定的地址，因此可以知道指向的數據在被Pin之後不能移動。與pin stack相反，可以知道數據將在對象的生命周期内固定。



1. 如果`T:Unpin` 則Pin<'a T>完全等同於`&'a mut T` Unpin意味著即使在Pin時也可以移動此類型，因此Pin對這種類型沒有影響。
2. 若`T:!Unpin` 則將`&mut T` 設置爲Pin T 需要unsafe
3. 大多數標準庫類型都是先了`Unpin`,  `async/.await`是個例外
4. 可以在每晚使用功能标志在类型上添加 !Unpin 绑定，或者在稳定版本将 std::marker::PhantomPinned
5. 可以將數據Pin到stack or heap
6. 將`!Unpin` 對象固定到 堆棧是unsafe的
7. 將`!Unpin`對象固定到heap 不需要unsafe，可以使用Box::Pin執行此操作
8. 对于 T: !Unpin 的固定数据，必须保持不变性，即从它被固定到调用 drop 的那一刻起，**它的内存不会失效或重新调整用途**。这是pin合同的重要组成部分。



## `Stream Trait`

Stream trait 類似於Future，但可以在完成之前產生多個值，類似於標準庫中的`itearator` trait

```rust
triat Stream{
    
    // stream 產生的值的類型
    type Item;
    
    //若未準備好則返回`Poll::Pending`,如果有值 則返回`Poll::Ready(Some(x))`, 若已經準備就緒，如果stream 已經完成，則為`Poll::Ready(None)`
    fn poll_next(self:Pin<&mut self>,ctx:&mut Context<'_>)->Poll<Oprion<Self::Item>>;
}
```

stream 常見實例是來自`futures` crate 的`channel` 類型的`receiver`。 每次從sender發送一個值時，都會產生Some(val),一旦`Sender` 被丟棄并且所有待處理的消息都已經收到，其將產生`None`。

```rust
use futures::executor::block_on;
use futures::{channel, SinkExt, StreamExt};

fn main() {
    println!("Hello, world!");
    let send_rec = send_recv();
    block_on(send_rec);
}

async fn send_recv(){
    const BUFFER_SIZE:usize =10;
    let (mut tx,mut rx)=channel::mpsc::channel::<i32>(BUFFER_SIZE);

   for i in 1..=10  {
        tx.send(i).await.unwrap();
    }

    drop(tx);  // 此處若是注釋掉這一行，程序及那個陷入死鎖當中，因爲一直在等待send，只有drop send 端時，receiver 才能接收到數據

    for i in 1..=10 {
        assert_eq!(rx.next().await,Some(i));
    }
}
```

## `iteration and concurrency`

類似於同步迭代器，有許多不同的方法來迭代和處理stream中的值（如 map、filter、fold）。但是 for 不能用於stream,可以使用`let` 和`next`。

```rust
async fn sum_with_next(mut stream: pin::Pin<&mut dyn Stream<Item=i32>>) -> i32 {
    let mut sum = 0;
    while let Some(item) =stream.next().await{
      sum +=item
    };
    sum
}
```



要同時處理stream中的多個元素，可以使用`for_each_concurrent` or `try_for_each_concurrent` 

```rust
async fn jump_around(
    mut stream: Pin<&mut dyn Stream<Item = Result<u8, io::Error>>>,
) -> Result<(), io::Error> {
    use futures::stream::TryStreamExt; // for `try_for_each_concurrent`
    const MAX_CONCURRENT_JUMPERS: usize = 100;

    stream.try_for_each_concurrent(MAX_CONCURRENT_JUMPERS, |num| async move {
        jump_n_times(num).await?;
        report_n_jumps(num).await?;
        Ok(())
    }).await?;

    Ok(())
}

```



## 一次執行多個`Future`

- `Join!` 等待所有的future完成
- `select!` 等待幾個future中的一個完成
- `Spawning` 創建一個頂級任務，該任務在環境中運行future以完成（unimpl）
- `FuturesUnordered` 一組產生每個子future結果的future（unimpl）

### Join!

可以等待多個不同的future完成同時執行他們

```rust
use futures::executor::block_on;
use futures::{channel, SinkExt, Stream, StreamExt, TryStreamExt};
use std::pin;

fn main() {
    println!("Hello, world!");
    // let send_rec = send_recv();
    // block_on(send_rec);
    let f = foos();
    block_on(f);
}

async fn send_recv() {
    const BUFFER_SIZE: usize = 10;
    let (mut tx, mut rx) = channel::mpsc::channel::<i32>(BUFFER_SIZE);

    for i in 1..=10 {
        tx.send(i).await.unwrap();
    }

    drop(tx);

    for i in 1..=10 {
        assert_eq!(rx.next().await, Some(i));
    }
    dbg!("send_recv");
}

async fn foo() {
    const BUFFER_SIZE: usize = 20;
    let (mut tx, mut rx) = channel::mpsc::channel::<i32>(BUFFER_SIZE);

    for i in 1..=10 {
        tx.send(i).await.unwrap();
    }

    drop(tx);

    for i in 1..=10 {
        assert_eq!(rx.next().await, Some(i));
    }
    dbg!("foo");
}

async fn foos(){
    let send_recv = send_recv();
    let foo = foo();
    futures::join!(send_recv,foo);

    dbg!("foos");
}
```

`Join!` 返回的值是一個元組，包含傳入的每個Future的輸出。 但是Join必須在`async fn`中使用

### `try_join!`

對於有回值的`future`,可以使用`try_join!` ,join! 會等待所有的future完成，即使出現錯誤。不同於`join!`  `try_join!` 遇到err時將會終止。

 

```rust
use futures::try_join;

async fn get_book() -> Result<Book, String> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book();
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}

```

注：future 傳遞給`try_join!` 必須都是具有相同的錯誤類型，建議使用`futures::future::TryFutureExt ` 中的`.map_err(|e|...)` 以及`.err_into()` 來合并錯誤類型

```rust
use futures::{
    future::TryFutureExt,
    try_join,
};

async fn get_book() -> Result<Book, ()> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book().map_err(|()| "Unable to get book".to_string());
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}

```



### `Select!`

同時運行多個future，允許在任何future完成後立即做出相應

```rust
async fn foo_select() {
   
    let t1 = send_recv().fuse();
    let t2 = foo().fuse();
    let t3= foos().fuse();

    futures::pin_mut!(t1,t2,t3);
    futures::select!{
        ()=t1=>println!("send ready"),
        ()=t2=>println!("foo ready"),
        ()=t3=>println!("foos ready"),
    }
}
```

>
>  fuse 創建一個迭代器，其會在遇到第一個None是返回
>     

`select`的基本語法 `<pattern> = <expression> => <code>`,

`select!` 還支持 `default` `complate`

若選中的分支沒有完成，則默認分支將運行，因此帶有默認分支的select 將始終立即返回，因爲其他的分支沒有完成時，`default` 將會運行

`complete` 分支可用於處理所有被選中的future都已完成且不在進行的情況，這在迭代`select!` 很方便

```rust
async fn foo_select() {

    let t1 = send_recv().fuse();
    let t2 = foo().fuse();
    // let t3= foos().fuse();

    futures::pin_mut!(t1,t2);
    futures::select!{
        ()=t1=>println!("send ready"),
        ()=t2=>println!("foo ready"),
        // ()=t3=>println!("foos ready"),
        complete=>unreachable!(), //never run
        default=>println!("default"),

    }
}

```

###  與 `Unpin` 和 `FusedFuture` 的交互

select! 中使用的future 必須同時實現`!Unpin` 和`Fused Future` trait,因此在select之前 調用 `async fn .fuse();`  以及`pin_mut();` 是很有必要的



**`Unpin` 是因爲`select!` 使用的future 不是按值獲取的，而是按照可變引用獲取的。通過不取得future的所有權，未完成的future 可以在調用future之後再次使用。**

類似的 `FusedFuture` 是因爲select不能再完成后輪詢future。`fusedfuture` 由追蹤他們是否已經完成的future實現。之輪詢完成的`future`。

注：Stream 具有相應的FusedStream trait 實現此trait或已使用`.fuse()` 的stream 將從`.next()\try_next()` 組合中產生`FusedFuture` future

```rust

#![allow(unused)]
fn main() {
use futures::{
    stream::{Stream, StreamExt, FusedStream},
    select,
};

async fn add_two_streams(
    mut s1: impl Stream<Item = u8> + FusedStream + Unpin,
    mut s2: impl Stream<Item = u8> + FusedStream + Unpin,
) -> u8 {
    let mut total = 0;

    loop {
        let item = select! {
            x = s1.next() => x,
            x = s2.next() => x,
            complete => break,
        };
        if let Some(next_num) = item {
            total += next_num;
        }
    }

    total
}
}

```

### 带有 `Fuse` 和 `FuturesUnordered` 的選擇循環中的并發任務

`Fuse::terminate()` 允許構造一個已經終止的空的`future` ,然後可以用需要的`future` 來填充，當需要在select loop 運行但在select loop 本身内部創建的任務時，十分便捷

 `.select_next_some()` 函数可以与 select 一起使用，只为从stream返回的 Some(_) 值运行分支，而忽略 Nones。

```rust

#![allow(unused)]
fn main() {
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) { /* ... */ }

async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let run_on_new_num_fut = run_on_new_num(starting_num).fuse();
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(run_on_new_num_fut, get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                // The timer has elapsed. Start a new `get_new_num_fut`
                // if one was not already running.
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                // A new number has arrived -- start a new `run_on_new_num_fut`,
                // dropping the old one.
                run_on_new_num_fut.set(run_on_new_num(new_num).fuse());
            },
            // Run the `run_on_new_num_fut`
            () = run_on_new_num_fut => {},
            // panic if everything completed, since the `interval_timer` should
            // keep yielding values indefinitely.
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}
}
```

當同一future的多個副本需要同時運行hi是，可以使用`FuturesUnorderd`類型。

```rust

#![allow(unused)]
fn main() {
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, FuturesUnordered, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) -> u8 { /* ... */ 5 }

// Runs `run_on_new_num` with the latest number
// retrieved from `get_new_num`.
//
// `get_new_num` is re-run every time a timer elapses,
// immediately cancelling the currently running
// `run_on_new_num` and replacing it with the newly
// returned value.
async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let mut run_on_new_num_futs = FuturesUnordered::new();
    run_on_new_num_futs.push(run_on_new_num(starting_num));
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                // The timer has elapsed. Start a new `get_new_num_fut`
                // if one was not already running.
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                // A new number has arrived -- start a new `run_on_new_num_fut`.
                run_on_new_num_futs.push(run_on_new_num(new_num));
            },
            // Run the `run_on_new_num_futs` and check if any have completed
            res = run_on_new_num_futs.select_next_some() => {
                println!("run_on_new_num_fut returned {:?}", res);
            },
            // panic if everything completed, since the `interval_timer` should
            // keep yielding values indefinitely.
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}

}

```





