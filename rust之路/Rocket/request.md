# request

一个route属性和功能的签名指定的内容必须时关于请求，称为路由处理程序

```rust
#[get("/")]
fn method(){
    /*
    do something
    */
}
```

以get的方式请求/路由，除了处理请求还有其他的事情：

- 动态路径段的类型
- 几个动态路径段的类型
- 传入正文数据的类型
- 查询字符串，表达和表单值得类型
- 请求得预期传入或传出格式
- 自定义得安全性或验证性策略

路由属性和功能签名协同工作以描述这些验证。Rocket的代码生成负责实际验证属性。

## methods

Rocket路由属性可以是`get`，`post`,`delete`,`head`

,`patch`,`option`都对应HTTP方法

```rust
#[post("/post")]
```

### head请求

`HEAD`当存在`GET`否则会匹配的路由时，Rocket会自动处理请求。它通过从响应中删除主体（如果有）来实现。还可以`HEAD`通过声明请求的路由来专门处理请求。Rocket不会干扰`HEAD`应用明确处理的请求。

### Reinterpretating

由于HTML表单只能作为`GET`或`POST`请求直接提交，因此Rocket在某些情况下会*重新解释*请求方法。如果`POST`请求包含的正文，`Content-Type: application/x-www-form-urlencoded`并且表单的**第一个**字段具有名称`_method`和有效的HTTP方法名称作为其值（例如`"PUT"`），则该字段的值将用作传入请求的方法。这允许Rocket应用程序提交非`POST`表单。

### 动态路由

在路由的路径中使用`< >`将变量名声明为动态路径段

```rust
#[get("/hi/<re>")]
fn index(re:&RawStr) -> String {
    format!("hello {}!",re.as_str())
}

fn main() {
    rocket::ignite().mount("/", routes![index]).launch();
}
```

> 允许任意数量的动态路径段。路径段可以是任何类型，包括自定义的类型，只要该类型实现了[`FromParam`](https://api.rocket.rs/v0.4/rocket/request/trait.FromParam.html)特征即可。将这些类型称为*参数保护*。Rocket实现`FromParam`了许多标准库类型以及一些特殊的Rocket类型。

```rust
#[get("/hello/<name>/<age>/<cool>")]
fn hello(name: String, age: u8, cool: bool) -> String {
    if cool {
        format!("You're a cool {} year old, {}!", age, name)
    } else {
        format!("{}, we need to talk about your coolness.", name)
    }
}
```

注： Rocket分别将*原始*字符串和解码后的字符串分开键入

> RawStr ：由Rocket提供的一种特殊类型，待变HTTP消息中为经过滤、未经验证和未经解码的原始字符串。
>
> 其存在就是为了验证字符串输入，通过类型通过类型，例如表示分离`String`，`&str`和`Cow<str>`，从未经验证的输入，由下式表示`&RawStr`。它还提供了有用的方法来将未验证的字符串转换为已验证的字符串。
>
> 由于RawStr已经实现了FormParam，所以其可以作为动态段的类型 ，

### 细节

通过`<param ..>`在路由中匹配多个路径，此类参数的类型被称为段防护，必须实现FormSegments，**必须是路径的最后**，其后的任何文本都将导致编译时错误。

```rust
use std::path::PathBuf;

#[get("/page/<path..>")]
fn get_page(path: PathBuf) { /* ... */ }
```

路径`/page/`将在`path`参数中可用。的`FromSegments`实现`PathBuf`确保`path`不会导致路径遍历攻击。这样，可以通过4行实现安全可靠的静态文件服务器：

```rust
use rocket::response::NamedFile;

#[get("/<file..>")]
fn files(file: PathBuf) -> Option<NamedFile> {
    NamedFile::open(Path::new("static/").join(file)).ok()
}
```

tip：Rocket 可以轻松的提供静态文件

> 需要通过Rocket应用程序提供静态文件，请考虑使用`StaticFiles`来自中的自定义处理程序`rocket_contrib`该处理程序非常简单：
>
> ```rust
> rocket.mount(&quot;/pubilc&quot;,StaticFile::from(&quot;/static&quot;))FF
> ```



### Forwarding

```rust
#[get("/hello/<name>/<age>/<cool>")]
fn hello(name: String, age: u8, cool: bool) { /* ... */ }
```

当参数类型不匹配时，Rocket*将*请求*转发*到下一个匹配的路由（如果有）。这一直持续到路由不转发请求或没有剩余路由可尝试为止。如果没有剩余路由，则会返回一个可自定义的**404错误**。

Rocket 以升序排列，选择从6到1的默认等级，路径等级可以使用rank属性设置。

```rust
#[get("/user/<id>")]
fn user(id: usize) { /* ... */ }

#[get("/user/<id>", rank = 2)]
fn user_int(id: isize) { /* ... */ }

#[get("/user/<id>", rank = 3)]
fn user_str(id: &RawStr) { /* ... */ }

fn main() {
    rocket::ignite()
        .mount("/", routes![user, user_int, user_str])
        .launch();
}
```

rank在参数user_int和user_str,请求过程

- 该`user`路线首先匹配。如果该`<id>`位置的字符串是无符号整数，则将`user`调用处理程序。如果不是，则将请求转发到下一个匹配的路由：`user_int`。
- 在`user_int`接下来的路由匹配。如果`<id>`是一个有符号整数，`user_int`则被调用。否则，将转发请求。
- 该`user_str`路由匹配最后。由于`<id>`始终为字符串，因此路由始终匹配。该`user_str`处理器将被调用。

tip：在发布过程中，路由的等级会显示在[]中。在应用程序启动期间在括号中找到路线的等级：`GET /user/<id> [3] (user_str)`。

> 当任何防护措施由于任何原因而失败时，包括参数防护措施，都可以在其位置使用`Option`或`Result`键入以捕获失败。
>
> 如果要忽略or路由中的`rank`参数，Rocket将发出错误并中止启动，这表明路由*发生冲突*，或者可以与相似的传入请求匹配。该参数可解决此冲突

### Default Ranking

如果未明确指定等级，则Rocket会分配默认等级。默认情况下，具有**静态路径和查询字符串的路由具有较低的等级（较高的优先级）**，而具有**动态路径和没有查询字符串的路由具有较高的等级（较低的优先级）**

| static path | query         | rank | example             |
| ----------- | ------------- | ---- | ------------------- |
| yes         | partly static | -6   | `/hello?world=true` |
| yes         | fully dynamic | -5   | `/hello/?<world>`   |
| yes         | none          | -4   | `/hello`            |
| no          | partly static | -3   | `/<hi>?world=true`  |
| no          | fully dynamic | -2   | `/<hi>?<world>`     |
| no          | none          | -1   | `/<hi>`             |

### Query String

可以使用路径段几乎相同的方式将查询段声明为静态或动态：

```rust
#[get("/hello?wave&<name>")]
fn hello(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}
```

`GET`给请求`/hello`具有的至少一个查询键`name`和的查询片段`wave`以任何顺序，忽略任何额外的查询片段。所述的值`name`查询参数被用作值`name`函数的参数。例如，请求`/hello?wave&name=John`将返回`Hello, John!`。可能导致相同响应的其他请求包括：

- `/hello?name=John&wave` （重新排序）
- `/hello?name=John&wave&id=123` （细分）
- `/hello?id=123&name=John&wave` （重新排序，额外的细分）
- `/hello?name=Bob&name=John&wave` （最后取值）

允许任意数量的动态查询段。查询段可以是任何类型，只要该类型实现了`FromFormValue`特征即可。

### Optional Parameters

查询参数允许*丢失*。只要请求的查询字符串包含路由的查询字符串的所有静态组成部分，该请求就会被路由到该路由。这允许使用可选参数，即使缺少参数也可以进行验证。

`Option<T>`用作参数类型。每当请求中缺少查询参数时，`None`都会将其作为值提供。使用的路线`Option<T>`如下所示：

```rust
#[get("/hello?wave&<name>")]
fn hello(name: Option<String>) -> String {
    name.map(|name| format!("Hi, {}!", name))
        .unwrap_or_else(|| "Hello!".into())
}
```

任何`GET`路径为`/hello`和`wave`查询段的请求都将路由到该路由。如果存在`name=value`查询段，则路由返回字符串`"Hi, value!"`。如果不存在`name`查询段，则路由返回`"Hello!"`。

就像类型的参数在查询中缺少该参数时`Option<T>`将具有值一样，类型`None`的参数在其缺失时`bool`也将具有值`false`。对于缺少参数的默认值可以定制自己的类型实现`FromFormValue`通过实现FromFormValue::default()。

### Multiple Segments

与路径一样，可以使用<param..> 与查询中的多个路由进行匹配，此参数的类型(称为查询保护)，必须实现了FromQuery trait，查询防护必须是查询的**最后组成部分**，查询参数之后的任何文本都将导致编译时错误。

查询保护器会验证所有其他不匹配的查询段(通过静态或动态查询参数)，或者手动实现FromQuery，

简而言之，这些类型允许使用具有命名字段的结构来自动验证查询/表单参数：

```rust
use rocker::request::Form;
#[derive(FromForm)]
struct User{
	name:String,
	account:usize,
}

#[get("/item?<id>&<user..>")]
fn item(id:usize,user:From<User>){/**/}
```

> 对于请求/ item？id = 100＆name = sandal＆account = 400的请求，上述项目路径将id设置为100，并将用户设置为User {名称：“ sandal”，帐户：400}。 要捕获无法验证的表单，请使用选项或结果类型：
>

### Request Guards

请求守卫是Rocket的工具之一，请求保护程序可以防止处理程序基于传入请求中包含的信息被错误调用。 更具体地说，请求防护是代表任意验证策略的类型。 验证策略是通过FromRequest特性实现的。 实现FromRequest的每种类型都是请求保护。

请求防护作为**处理程序的输入**出现。 任意数量的**请求卫士**可以作为**参数**出现在**路由处理程序**中。 **在调用处理程序之前，Rocket将自动为请求保护措施调用FromRequest实现。** Rocket仅在其所有守卫通过后才将请求分发给处理程序。

```go
#[get("/<param>")]
fn index(param:isize,a:A,b:B,c:C){}
```

请求守卫按照从左至右的声明顺序开始, 例中从A——> B  ——> C，如果请求失败，则会尝试匹配下一个。

### Custom Guards

可以为自定义的类型实现FromRequest，

例如，为防止敏感路由运行，除非请求标头中没有ApiKey，可以创建一个实现FromRequest的ApiKey类型，然后将其用作请求保护：

```rust
#[get("/sensitive")]
fn sensitive(key: ApiKey) { /* .. */ }
```

> 可以为AdminUser类型实现FromRequest，以使用传入的cookie对管理员进行身份验证。 然后，确保在其参数列表中具有AdminUser或ApiKey类型的任何处理程序仅在满足适当条件的情况下才被调用。

### Forwarding Guards

```rust
use rocket::response::{Flash, Redirect};

#[get("/login")]
fn login() -> Template { /* .. */ }

#[get("/admin")]
fn admin_panel(admin: AdminUser) -> &'static str {
    "Hello, administrator. This is the admin panel!"
}

#[get("/admin", rank = 2)]
fn admin_panel_user(user: User) -> &'static str {
    "Sorry, you must be an administrator to access this page."
}

#[get("/admin", rank = 3)]
fn admin_panel_redirect() -> Redirect {
    Redirect::to(uri!(login))
}
```

> 上面的三个路由对身份验证和授权进行编码。 仅当管理员登录后，admin_panel路由才会成功。然后，才会显示管理面板。 如果用户不是管理员，则AdminUser防护将转发。 由于admin_panel_user路由的排名次高，因此将尝试下一个。 如果有任何用户登录，则此路由成功，并显示授权失败消息。 最后，如果未登录用户，则尝试使用admin_panel_redirect路由。 由于此路线没有警卫，因此它总是成功的。 用户将被重定向到登录页面。
>

### cookies

Cookies是重要的内置请求防护程序：它使您能够获取，设置和删除Cookies。 因为Cookies是请求保护，所以可以将其类型的参数添加到处理程序中

```rust
use rocket::http::Cookies;
#[get("/")]
fn index(cookie:Cookies)->Option<String>{
    cookies.get("message")
    		.map(|value| format!("Message:{}",value))
}
```

> 这导致可以从处理程序访问传入请求的cookie。 上面的示例检索一个名为message的cookie。 也可以使用Cookies卫士设置和删除Cookies。 
>

### Private Cookies

通过`Cookies::add()` 方法添加的Cookies设置明文。换言之，该直对客户端可见，  对于敏感数据，Rocket提供了专用cookie。

转用cookie类似于常规的cookie，不同之处在于它们**使用经过身份验证的加密**（一种同时提供机密性，完整性和真实性的加密形式）进行加密，

这意味着客户端无法检查，篡改或制造私人cookie，可以将私有cookie视为已签名和加密的。

> 检索，添加和删除私有cookie的API相同，除了方法使用_private后缀。 这些方法是：get_private，add_private和remove_private。 

```rust
use rocket::http::{Cookie,Cookies};
use rocket::response::{Flash,Redirect};

///retrieve the user ID if any
#[get("/user_id")]
fn user_id(mut cookies:Cookies)->Option<String>{
    cookies.get_private("user_id")
    		.map(|cookie| format!("User id {}",cookie.value))
}


/// remove the use_id cookie
#[post("/")]
fn logout(mut cookies:Cookies)->Flash<Redirect>{
	cookies.remove_private(Cookie::named("user_id"));
    Flash::success(Redirect::to("/"),"Successfully logged out")	
}
```

在构建时，可以通过禁用Rocket的默认功能，进而禁用默认的private-cookies功能，来取消对依赖于环形库的私有cookie的支持。 

```rust
[dependencies]
rocket = { version = "0.4.5", default-features = false }
```

### Secret Key

为了加密私有cookie，Rocket使用secret_key配置参数中指定的256位密钥。 **如果未指定，Rocket将自动生成一个新密钥。**

注：只能使用与加密相同的密钥来解密专用cookie。

在使用私有cookie时，设置**secret_key**配置参数很重要，这样cookie才能在应用程序重新启动后正确解密。 如果应用程序在生产中运行而没有配置secret_key，则Rocket会发出警告。

通常通过诸如openssl之类的工具来生成适合用作secret_key配置值的字符串。 使用openssl可以使用openssl rand -base64 32命令生成256位base64密钥。

### One-At-A-Time

出于安全原因，**Rocket目前要求一次最多激活一个Cookies实例**。 遇到这种限制并不常见，但是如果出现这种限制，可能会造成混乱

如果发生这种情况，Rocket将向控制台发出如下消息：

```
=> Error: Multiple `Cookies` instances are active at once.
=> An instance of `Cookies` must be dropped before another can be retrieved.
=> Warning: The retrieved `Cookies` instance will be empty.
```

调用违规处理程序时，将发出消息。 通过确保由于有问题的处理程序而无法同时激活两个Cookie实例，可以解决此问题。 常见的错误是使处理程序使用Cookies请求防护，以及使用自定义请求防护来检索Cookies，

```rust
#[get("/")]
fn bad(cookies: Cookies, custom: Custom) { /* .. */ }
```

因为cookie防护将在自定义防护之前触发，所以当cookie的实例已经存在时，自定义防护将检索cookie的实例。 只需**交换防护的顺序即可解决此情况**

```rust
#[get("/")]
fn good(custom: Custom, cookies: Cookies) { /* .. */ }
```

当使用按需修改Cookie的请求防护（例如FlashMessage）时，会发生类似的问题。 在这种情况下，解决方法是**在访问FlashMessage之前删除Cookies实例。**

```rust
use rocket::request::FlashMessage;

#[get("/")]
fn bad(cookies: Cookies, flash: FlashMessage) {
    // Oh no! `flash` holds a reference to `Cookies` too!
    let msg = flash.msg();
}

#[get("/")]
fn good(cookies: Cookies, flash: FlashMessage) {
    std::mem::drop(cookies);

    // Now, `flash` holds an _exclusive_ reference to `Cookies`. Whew.
    let msg = flash.msg();
}
```

### Format

路由可以使用format route参数指定它接收或者响应的数据格式， 参数的值是一个字符串，用于标识HTTP媒体类型或简写形式

对于JSON数据，可以使用字符串application / json或仅使用json。

当路由指示有效负载支持方法（PUT，POST，DELETE和PATCH）时，格式路由参数会指示Rocket检查传入请求的Content-Type标头。 只有Content-Type标头与format参数匹配的请求才会与路由匹配。

```rust
#[post("/user", format = "application/json", data = "<user>")]
fn new_user(user: User) { /* ... */ }
```

> post属性中的format参数声明只有Content-Type：application / json的传入请求将与new_user匹配。 
>
> 多数常用的格式参数也支持简写。 除了使用完整的Content-Type，
>
> format=“ application / json” 简写为：format =“ json”。

```rust
#[get("/user/<id>", format = "json")]
fn user(id: usize) -> User { /* .. */ }
```

get属性中的format参数声明，只有具有application / json作为Accept头中的首选媒体类型的传入请求才能与用户匹配。 如果相反，该路线已被声明为发布，则Rocket会将格式与传入响应的Content-Type标头进行匹配。

### Body Data

像Rocket的许多其他内容一样，Body数据处理也是类型定向的。 要指示处理程序需要主体数据，请使用data =“ <param>”对其进行注释，

其中param是处理程序中的参数，参数的类型必须实现FromData trait

```rust
#[post=("/",data="<input>")]
fn new(input:T){}
```

任何实现FromData的类型也称为数据保护

### Forms

表单是Web应用程序中最常见的数据类型之一，而Rocket使其易于处理。

```rust
use rocket::request::Form;

#[derive(FromForm)]
struct Task{
    complets:bool,
    description:String,
}

#[post=("/todo",data="<task>")]
fn new(task:Form<Task>){}
```

Form类型需要实现FromForm trait， Form类型就实现FromData特征

> 实例中 自动为Task 派生了 FromForm trait，可以为字段实现FromFormValue的任何结构派生FromForm
>
> 如果POST / todo请求到达，则表单数据将自动解析为Task结构。 如果到达的数据不是正确的Content-Type，则转发请求。 如果数据无法解析或仅是无效的，则返回可自定义的400-错误请求或422-无法处理的实体错误。 可以通过使用Option和Result类型来捕获转发或失败

```rust
#[post("/todo", data = "<task>")]
fn new(task: Option<Form<Task>>) { /* .. */ }
```

### Lenient Parsing

默认情况下，Rocket的FromForm解析是严格的。仅当Form <T>包含T中确切的字段集时，Form <T>才能从传入的表单中成功解析

Form <T>在缺少和/或多余的字段时将**出错**。

通过LenientForm数据类型选择退出此行为。只要表单包含T中字段的超集，LenientForm <T>就会从传入的表单中成功解析。

**LenientForm <T>自动无误地丢弃多余的字段**。

可以在任何使用Form的地方使用LenientForm。实现FromForm也需要它的通用参数。

```rust
use rocket::request::LenientForm;
#[derive(FromForm)]
struct Task{}

#[post=("/todo",data="<task>")]
fn new(task:LenientForm<Task>){}
```

### Field Renaming

一般情况下，Rocket将传入的表单字段名称和结构体字段名称相匹配，

使用`#[form(field="name")]`字段进行批注，指定相对应字段的名称

```rust
#[derive(FromForm)]
stcurt From_data{
    #[form(field="type")]
    api_type:String
}
```

Rocket 将会把表单中type类型的数据 匹配到结构体api_type之中

### Field Validation

表单字段可以通过`FromFormValue` 特性的实现验证

对表单中的年龄进行认证

```rust
use rocket::http::RawStr;
use rocket::request::FromFormValue;

struct AdultAge(usize);
impl<'v'> FromFormValue<'v> for AdultAge{
    type Error=&'v RawStr;
    fn from_form_value(form_value: &'v RawStr) -> Result<AdultAge, &'v RawStr> {
        match form_value.parse::<usize>() {
            Ok(age) if age >= 21 => Ok(AdultAge(age)),
            _ => Err(form_value),
        }
    }
}

#[derive(FromForm)]
struct Person{
    age:AdulAge
}
```

如果提交的年龄错误，Rocket不会调用需要该结构有效表单的处理程序。

使用Option 或者Result类型用于字段以捕获解析失败。

```rust
#[derive(FromForm)]
struct Person{
    age:Option<AdultAge>
}
```

可以为具有无效字段的**枚举**派生FromFormValue特性

```rust
#[derive(FromFormValue)]
enum Val{
    F,
    S,
    T,
}
```

派生为装饰的枚举生成FromFormValue特性的实现。 当表单值（不区分大小写）匹配变体名称的字符串化版本时，实现成功返回，并返回该变体的实例

### Json

处理json数据，需要使用rocket_contrib中的json类型

```rust
use serde::Deserialize;
use rocket_contrib::json::Json;

#[derive(Deserialize)]
struct Task{
    description:String,
    conplete:bool,
}

#[post=("/todo",data="<task>")]
fn new(task:Json<Task>){}
```

唯一的条件是**Json中的泛型类型实现了Serde的Deserialize特性**

### Streaming

直接处理传入的数据， Rocket通过Data类型完成需求

```rust
use rocket::Data;

#[post=("/upload",format="plain",data="<data>")]
fn upload(data:Data)->Result<String,std::io::Error>{
    data.stream_to_file("/tmp/upload.txt").map(|n|n.to_string())
}
```

> text / plain传入的数据被流传输到tmp / upload.txt，如果上传成功，则写入的字节数将作为纯文本响应返回。 。 如果上传失败，则返回错误响应。

tip:读取传入数据时，始终设置限制

> 为了防止DoS攻击，您应该限制您愿意接受的数据量。 take（）阅读器适配器使此操作变得容易：data.open().take(LIMIT)

### Error Catchers

路由失败的原因：

- 守卫失败
- 处理程序返回失败的响应者
- 没有匹配的路线

若发生以上任意一种情况，Rocket都会相客户端返回错误。

为此Rocket调用与错误的状态代码相对应的捕获器，Catcher和routes类似，区别在于

- 捕捉程序仅在错误条件下被调用。
- 捕捉器使用catch属性声明。
- 捕捉器使用register（）而不是mount（）进行注册。
- 对Cookie的任何修改都将在调用捕获器之前清除。
- 错误捕获器无法调用任何类型的防护。

> Rocket为所有标准HTTP错误代码提供了默认捕获器。 要覆盖默认捕获程序，或为自定义状态代码声明捕获程序，请使用`catch`属性，该属性采用与HTTP状态代码相对应的单个整数进行捕获。

如：声明404错误的捕获器

```rust
use rocket::Request;

#[catch(404)]
fn no_found(req:Request){}
```

与路由一样，返回类型（此处为T）必须实现Responder。 

```rust
#[catch(404)]
fn not_found(req: &Request) -> String {
    format!("Sorry, '{}' is not a valid path.", req.uri())
}
```

和路由一样。Rocket需要对catch进行注册，然后才能用它来捕获错误，

通过捕获器用捕获器列表调用register（）catchers！ 宏

```rust
fn main(){
	rokcet::ignite().register(catchers![not_found]);
}
```

与路由请求处理程序不同，**捕获器仅使用零或一个参数。 如果捕获器接受参数，则它必须是＆Request类型**。

















