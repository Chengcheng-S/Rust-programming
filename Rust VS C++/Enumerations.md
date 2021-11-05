## Enumerations

### C/C++

在C++中`enum` 是一堆被赋予int值得标签，其本质上是以堆带有标量值得常数。

```c++
enum HttpResponse{
	okay=200,
    not_found=404,
    internal_err=500,
}
```

C++11 拓展了原先的用法，允许使用字符来存储数据

```c++
enum LibraryCode:char{
    checked_in='I',
    checked_out='o',
    checked_out_late='L'
};
```

### Rust

Rust中的`enum`类似于c++中

```rust
enum HttpRespose{
	OK=200,
    NotForund=404,
    InternalError=500
}
```

枚举可以保存其他类型的数据，因此可以传递比静态值本身多得多的值

```rust
enum  HttpResponse{
	OK,
    NotFounf(String),
    InternalError(String,String,Vec<u8>)
}
```

可以为枚举绑定方法

```rust
impl HttpResponse{
	pub fn	code(&self)->{
        match *self{
            HttpResponse::OK=>200,
            HttpResponse::NotFound(_)=>404,
            HttpResponse::InternalError(_,_,_)=>500,
        }
    } 
}
```