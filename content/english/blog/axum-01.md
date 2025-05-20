+++
title = "使用 axum 构建微服务（-）"
date = 2025-04-29T17:00:00+08:00
tags = ["rust", "axum"]
categories = ["rust"]
draft = false
IMAGE = "/images/pl-banner.jpg"
AUTHOR = "LW"
+++

一直想使用 `axum` 完成工作中的一些服务功能，发现版本已经更新到了 `0.8.x` 重新梳理并学习一步一实现一个小的功能


## 开始 {#开始}

> axum is a web application framework that focuses on ergonomics and modularity

-   先根据 `example` 启动一个 web 服务，基于 `tokio` and `hyper`

<!--listend-->

```rust
use axum:: {
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    // build application
    let app = Router::new().route("/", get(|| async {"Hello, world"}));

    // run app with hyper,listeing on 3000
    let listener = tokio::net::Tcplistenser::bind("0.0.0.0:3000").await.unwrap();
    axum::server(listener, app).await.unwrap();
}

```

-   cargo.toml

<!--listend-->

```shell

# 启用所有功能
cargo add tokio --features full

# 按需
cargo add tokio --features macros, rt-multi-thread
```


## Router {#router}

用于路由映射

-   路由定义

<!--listend-->

```rust
use axum::{Router, routing::get};

// our router
let app = Router::new()
    // "/" Get 请求 调用 root function
    // return HTTP status code 200
    .route("/", get(root))
    // "/foo" Get 请求，调用 get_foo function, POST 请求, 调用 post_foo function
    .route("/foo", get(get_foo).post(post_foo))
    .route("/foo/bar", get(foo_bar));

// function 为 async
async fn root(){}
```

-   Handlers 处理器
    路由处理器是一个 “异步函数” ， 可以接收一个或者多个 “提取器” 并且可以转换为需要的内容
    类似与 `springboot` 中的 `Controller`
    ```java
    @RestController
    class AppController {

    @Get("/")
    public void root() {
        system.out.println("xxxxx");
    }
        ....
    }

    ```

    -   参数 不超过 16个，最后一个实现 `FromRequest` 其余参数要实现 `FromRequestParts`

    -   参数 必须实现 `Send`

    -   响应 要实现 `IntoResponse`

-   Extractors 提取器
    提取器也是实现了 `FromRequest` 和 `FromRequestParts` 的类型
    跟我我们的需求，一般会：
    -   获取query 参数

<!--listend-->

```rust

async fn query(Query(params): Query<HashMap<String, String>>)

```

-   获取 Path 参数 并获取 动态 参数 "/users/{id}"

<!--listend-->

```rust


async fn path(Path(uid): Path<u32>) {}

```

-   获取 headers 数据

<!--listend-->

```rust
async fn headers(headers: HeaderMap){}

```

-   获取 body 体参数 一般是 JSON

<!--listend-->

```rust
// raw reqyest body
async fn rawString(body:String) {}

```

```rust

use serde::Deserialize;

#[derive(Deserialize)]
struct CreateUserReq {
    name: String,
    Passwd: String,
}

async fn create_user(Json(payload): Json<CreateUserReq>)

```
