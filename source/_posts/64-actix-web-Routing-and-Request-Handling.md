---
title: '64.actix_web: Routing and Request Handling'
date: 2024-03-23 16:46:13
tags:
  - Rust
  - Web 开发
  - Actix
categories: Actix
---
# 什么是路由 - Routing
在`actix-web` 中，路由定义了程序执行哪一段代码来回应不同`URL` 的请求。

在`actix-web` 中，定义路由也就定义了哪个函数来处理哪一个`URL`。

## 定义路由
### 在`HttpServer` 实例中定义路由
```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
	HttpServer::new(|| {
		App::new()
			.service(hello)
			.route("/hey", web::get().to(manual_hello))
	})
	.bind(("127.0.0.1", 8080))?
	.run()
	.await
}
```
在上面的代码中，可以看到`main` 函数中，定义的`App` 实例，可以通过`route` 函数来注册路由。
上面的代码定义一个路由`/hey`，当客户端使用`get` 请求访问这个`URL` 时，调用`manual_hello` 这个函数来回应。

### 使用marco 宏定义路由
```rust
#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello World!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}
```

可以看到，每个函数的上面都有一个`actix-web` 内置的宏附加了路由信息。

在`main` 函数的`App` 示例中通过`service` 函数来注册服务。
```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
	HttpServer::new(|| {
		App::new()
			.service(hello)
			.service(echo)
	})
	.bind(("127.0.0.1", 8080))?
	.run()
	.await
}
```