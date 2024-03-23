---
title: '65.actix_web: Handling Requests'
date: 2024-03-23 16:51:10
tags:
  - Rust
  - Web 开发
  - Actix
categories: Actix
---
# Request Handler - 请求处理程序
请求处理程序是一个异步函数能够从客户端请求中提取参数(`impl FromRequest`)，并且能返回可以被转换成`HttpResponse` 的数据类型(`impl Responder`)。

请求处理会发生在两个阶段：
1. `Handler` 函数(对象)被调用，返回实现了`Responder` 特征的任意对象。
2. `respond_to()` 函数被上一步返回的对象调用，该函数将该对象转换为`HttpResponse` 或`Error`。

# Type-safe Information Extraction
`Extractor` 是`actix-web` 中一个功能强大的工具，能够帮助你轻易地处理和提取从客户端发来的请求中的数据，让程序员专注在逻辑处理上而不是如何提取数据上。另一个值得一提的点是，`Extractor` 是一个类型安全的信息提取器，提取的任何数据都需要提前设置好它的类型。

`actix-web` 提供了多种类型的`extractors` 来适用于不同场景。

## Path
`Path` 是用来从请求路径(URL) 中提取数据的提取器。路径(URL) 中可以被提取的部分被称为"动态片段 - dynaimc segments"，`actix-web` 使用`{}` 来标记。程序员可以从路径(URL) 中反序列化任意的变量。

```rust
use actix_web::{get, web, App, HttpServer, Result};

// extract path info from "/users/{user_id}/{friend}" url
// {user_id} - deserializes to a u32
// {friend} - deserializes to a String
#[get("/users/{user_id}/{friend}")]
async fn index(path: web::Path<(u32, String)>) -> Result<String> {
	let (user_id, friend) = path.into_inner();
	Ok(format!("Welcome {}, user_id is {}", friend, user_id))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
	HttpServer::new(|| App::new().service(index))
				.bind(("127.0.0.1", 8080))?
				.run()
				.await
}
```

在上面的例子中，请求处理程序可以处理请求形态为`/users/user_id<u32>/friend<String>` 的URL。请求处理程序使用`into_inner()` 方法将`URL` 中定义的`user_id` 和`friend` 的数据赋值给一个元组（这里的元组会按函数体参数部分声明的顺序）。
### 匹配动态片段名提取数据
可以通过匹配动态数据名和`Struct` 字段名，来提取数据，这里的`struct` 需要实现来自`serde` 的`Deserialize` 的特征。

```rust
use actix_web::{get, web, Result};
use serde::Deserialize;

#[derive(Deserialize)]
struct Info {
    user_id: u32,
    friend: String,
}

// 定义path extract
#[get("/users/{user_id}/{friend}")]
async fn index(info: web::Path<Info>) -> Result<String> {
    Ok(format!(
	    "Welcome {}, user_id {}",
	    info.friend, info.user_id
    ))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{App, HttpServer};

	HttpServer::new(|| App::new().service(index))
				.bind(("127.0.0.1", 8080))?
				.run()
				.await
}
```

### 非类型安全的一种变体
```rust
#[get("/users/{user_id}/{friend}")] // <- define path parameters
async fn index(req: HttpRequest) -> Result<String> {
    let name: String = req.match_info().get("friend").unwrap().parse().unwrap();
    let userid: i32 = req.match_info().query("user_id").parse().unwrap();

    Ok(format!("Welcome {}, user_id {}!", name, userid))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{App, HttpServer};

    HttpServer::new(|| App::new().service(index))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

## Query
用于从请求的`query` 中提取信息。针对类似的`URL` 形态，`/?id=64&response_type=Code`, `Query` 可以提取键为`id` 和`response_type` 的值。

[Struct actix_web::web::Query](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Query.html):
1. [method - from_query](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Query.html#method.from_query)
2. [method - into_inner](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Query.html#method.into_inner)


```rust
use actix_web::{get, web};
use serde::Deserialize;

#[derive(Debug, Deserialize)]
pub enum ResponseType {
	Token,
	Code,
}

#[derive(Debug, Deserialize)]
pub struct AuthRequest {
	id: u64,
	response_type: ResponseType,
}

// Deserialize `AuthRequest` Struct from query string.
// This handler gets called only if the request's query parameters contain both fields.
#[get("/")]
async fn index(info: web::Query<AuthRequest>) - String {
	format!("Authorization request for id = {} and type = {:?}!", info.id, info.response_type)
}

#[get("/debug1")]
async fn debug1(info: web::Query<AuthRequest>) -> String {
    dbg!("Authorization object = {:?}", info.into_inner());
    "Ok".to_string()
}
//[src/main.rs:38:5] "Authorization object = {:?}" = "Authorization object = {:?}"
//[src/main.rs:38:5] info.into_inner() = AuthRequest {
//    id: 64,
//    response_type: Code,
//}

#[get("/debug2")]
async fn debug2(info: web::Query<AuthRequest>) -> String {
    dbg!("Authorization object = {:?}", info);
    "Ok".to_string()
}
//[src/main.rs:44:5] "Authorization object = {:?}" = "Authorization object = {:?}"
//[src/main.rs:44:5] info = Query(
//    AuthRequest {
//        id: 64,
//        response_type: Code,
//    },
//)
```

可以通过[QueryConfig](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.QueryConfig.html)配置提取器进程。
## Json
`Json` 作为提取器能够将请求体反序列化为`struct`（`struct` 必须实现`serde::Deserialize`）。，可以使用[JsonConfig](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.JsonConfig.html)配置提取器进程。

[Struct actix_web::web::Json](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Json.html):
1. [method - into_inner()](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Json.html#method.into_inner)

```rust
use actix_web::{post, web, App};
use serde::Deserialize;

#[derive(Deserialize)]
struct Info {
	username: String,
}

#[post("/")]
async fn index(info: web::Json<Info>) -> String {
	format!("Welcome {}!", info.username)
}
```

## URL-Encoded Forms
这个请求处理器仅在请求内容为`application/x-www-form-urlencoded` 格式时生效。可以使用[FormConfig](https://docs.rs/actix-web/4/actix_web/web/struct.FormConfig.html) 配置。

[Struct actix_web::web::Form](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Form.html):
1. [method - into_inner()](https://docs.rs/actix-web/4.5.1/actix_web/web/struct.Form.html#method.into_inner)

```rust
use actix_web::{post, web, App};
use serde::Deserialize;

#[derive(Deserialize)]
struct Info {
	name: String,
}

// - request header declare the content type is `application/x-www-form-urlencoded`
// - request payload deserializes into `Info` struct
#[post("/")]
async fn index(form: web::Form<Info>) -> Result<String> {
	Ok(format!("Welcome {}!), form.name)
}
```

## Others
Other extractors:
+ [Data](https://docs.rs/actix-web/4/actix_web/web/struct.Data.html) - accessing pieces of application state.
+ [HttpRequest](https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html) - `HttpRequest` is itself an extractor, in case you need access to other parts of the request.
+ String - can convert a request's payload to a String. [Example](https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html#impl-FromRequest-for-String)
+ [Bytes](https://docs.rs/actix-web/4/actix_web/web/struct.Bytes.html) - can convert a request's payload into Bytes. [Example](https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html#impl-FromRequest-5)
+ [Payload](https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html) - Low-level payload extractor primarily for building other extractors. [Example](https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html)