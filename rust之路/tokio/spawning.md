## spawning

TCP套接字  `tokio::net::TcpListener`

TcpListener绑定端口6379，然后在loop中接收套接字，每个套接字都经过处理然后关闭。

```rust
use tokio::net::{TcpListener,TcpStream};
use mini_redis::{Connection,Frame};

#[tokio::main]
async fn main(){
    // bind listener to the address
    let listener=TcpListener::bind("127.0.0.1:6739").await.unwrap();
    
    loop{
        let(socket,_)=listener.accept().await.unwrap();
        process(socket).await;
       
    }
    
}
async fn process(socket::TcpStream){
	let mut connection=Connection::new(stocket);
    
    if let Some(frame)=connection.read_frame().await.unwrap(){
        println!("GOT! {:?}",frame);
 		let response=Frame::Error("unimplemented".to_string());
        connection.write_frame(&response).await.unwrap();
    }
    
}
```

### Concurrency

```rust
async fn reads() {
    let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();

    loop {
        let (socket, _) = listener.accept().await.unwrap();
        // process(socket);
        thread::spawn(async move {
            process(socket).await;
        });
    }
}
```

