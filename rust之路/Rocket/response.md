# response

## responder

实现Responder的类型知道如何从其值生成响应。 响应包括HTTP状态，标头和正文。 主体可以是固定大小的，也可以是流式的。 给定的Responder实现决定使用哪个。 例如，String使用固定大小的主体，而File使用流式响应。 响应者可以根据他们正在响应的传入请求动态调整其响应。

### Wrapping

```rust
struct WrappingResponder<R>(R);
```

R是实现响应者的某种类型，

包装响应程序在同一响应进行之前会修改R返回的响应。

Rocket在状态模块中提供了响应程序，这些响应程序将覆盖已包装的响应程序的状态代码。 

```rust
use rocket::response::status;
#[post("/<id>")]
fn new(id:usize)->status::Accepted<String>{
	status::Accepted(Some(format!("id:{}",id)))	
}
```

同样，内容模块中的类型可用于覆盖响应的内容类型,例如：&‘static str的Content-Type设置为Json，可以使用`content::Json`类型

```rust
use rocket::response::Content;

#[get("/")]
fn json()->Content::Json<&‘static str>{
    content::Json("{'one':'the first message'}")
}
```

注：这与rocket_contrib中的json不同

### Errors

响应程序可能会失败，返回带有给定状态的Err，这种情况下，rocket将请求转发给给定状态码的错误捕获器。

- 如果已经为给定的状态代码注册了错误捕捉器，Rocket将调用它。catcher创建并向客户端返回响应。
- 如果没有注册错误捕获器，并且错误状态代码是标准HTTP状态代码之一，则将使用默认错误捕获器。

默认错误捕捉器返回一个带有状态代码和描述的HTML页面。如果自定义状态代码没有捕捉器，Rocket将使用500错误捕捉器返回响应。

### Status

通过直接返回状态来手动将请求转发给捕获器

例如：转发给406的catcher Not Acceptable

```rust
use rocket::http::Status;
#[get("/")]
fn just_fail()->Status{
    Status::NotAcceptable
}
```

状态生成的响应取决于状态码本身，对于错误状态码，状态转发到相应的错误捕获器

| Status Code Range |       Response       |
| :---------------: | :------------------: |
|     [400,599]     | 为给定状态转发给捕手 |
|  100，[200,205]   |     给定状态为空     |
|    其他状态码·    |         无效         |



### Custom Responders

> Responder trait文档详细说明了如何通过显式实现trait来实现自定制响应器。然而，对于大多数用例，Rocket可以自动派生出应答器的实现。特别是，如果自定义响应程序包装现有的响应程序、标头或设置自定义状态或内容类型，则可以自动派生响应程序

```rust
use rocket::http::{Header,ContentType};

#[derive(Responder)]
#[response(status=500,content_type="json")]
struct MyResponder{
    inner:OtherResponder,
    header:ContentType,
    more:Header<'static'>,
    uunrelated:MyType,
}
```

Rocket生成了一个响应程序实现：

- 将响应的状态设置为500
- 内容设置为了 json
- 添加标题 self.header self,more
- 完成响应self.inner

注：第一个字段用作内部响应程序，而所有**剩余字**段（除非使用#[response（ignore）]忽略）都作为头**添加到响应**中。可选的#[response]属性可用于自定义响应的状态和内容类型。因为ContentType和Status本身就是标头，所以您还可以通过简单地包含这些类型的字段来动态设置内容类型和状态。

### Implementations

Rocket 在Rust标准库中为String，&str，File，Option和Result许多类型实现了Responder，

#### String

&str和String的Responder实现：将字符串用作sized body，并将响应的Content-Type设置为text/plain

```rust
use std::io::Cursor;

use rocket::request::Resquest;
use rocket::response::{self,Response,Responder};

impl<'a> Responder<'a> for String{
    fn respond_to(self,_:&Request)->response::Result<'a>{
        Response::build()
        		.header(ContentType::Plain)
        		.sized_body(Cursor::new(self))
        		.ok()
    }
}
```

为String实现Responder

```rust
#[get("/string")]
fn handler()->&'static str{
    "do something"
}
```

#### Option

Option包装响应程序：Option<T>仅在T实现响应程序时才返回，若Option为Some，则使用包装响应来响应客户端。否则，将向客户端返回错误404-Not Found

这类实现使得Option 成为一种方便的类型，可以在处理时是否直到内容是否存在之前返回他。

##### 例：

Option，我们可以实现一个文件服务器，该文件服务器在仅有的4条惯用语行中，当找到文件时返回200，而在找不到文件时返回404：

```rust
use rocket::response::NamedFile;

#[get("/<file..>")]
fn file(file::PathBuf)->Option<NameFile>{
    NameFile::open(path::new("/static")).join().ok()
}
```

#### Result

结果是一种特殊的包装响应程序：其功能取决于错误类型E是否实现响应程序。

错误类型E实现Responder时，可以使用Ok或Err中包装的Responder，来响应客户端，

**这意味着可以在运行时动态选择响应者，并且可以根据情况使用两种不同类型的响应。**

##### 例

```rust
use rocket::response::NamedFile;
use rocket::response::status::NotFound;

#[get("/<file..>")]
fn files(file:PathBuf)->Result<NamedFile,NotFound<String>>{
    let path=Path::new("static/").join(file);
    NameFild::open(&path)map_err(|e| NotFound(e.to_string()))
}
```

若错误类型E未实现Responder，则使用其Debug实现将错误简单地记录到控制台，并将500错误返回给客户端。

## Rocket Responders

响应程序 在相应模块和rocket_contrib:

- Content - 用于覆盖响应的Content-Type
- NameFile- 文件流到客户端，根据文件的扩展名 自动设置Content-Type
- Readirect -  客户端重定向到其他的URI
- Stream - 将响应从任意读取器类型流式传输到客户端。
- status - 包含重写响应的状态代码的类型
- Flash - 设置访问时删除的“闪存”cookie
- Json -  自动将值序列化为Json
- MsgPack - 自动将值序列化到MessagePack中
- Template - 使用手柄或Tera渲染动态模板

### strem

流类型：当需要向客户端发送大量数据时，最好将数据流式传输到客户端，避免消耗大量内存，Rocket提供了流类型，流类型可以从任何读取类型创建：

```rust
use std::os::unix::net::UnixStream;

use rocket::response::Stream;

#[get("/stream")]
fn stream()->Result<Stream<UnixStream>,std::io::Error>{
    UnixStream::connect("/path/to/my/socket").amp(Stream::from)
}
```











