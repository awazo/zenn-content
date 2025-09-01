---
title: "axum の Router をエンドポイントの階層で分けて書く"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "Rust", "axum", "Router", "nest" ]
published: true
---

# axum の Router ってどう書く？
Rust のウェブアプリケーションフレームワーク [axum](https://docs.rs/axum/latest/axum/) で、エンドポイントを用意するとき、[Router 構造体](https://docs.rs/axum/latest/axum/struct.Router.html) を使用するのだけれど、よくあるサンプルコードは以下みたいなものだったりする。

```Rust
use axum::{
    http::StatusCode,
    response::IntoResponse,
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    let router = Router::new()
        .route("/", get(root));

    let listener
        = tokio::net::TcpListener::bind("0.0.0.0:8888").await.unwrap();
    axum::serve(listener, router).await.unwrap();
}

async fn root() -> impl IntoResponse {
    (StatusCode::OK, "hello!".to_string())
}
```

`async fn main() {` の中で `Router::new()` して、あとはそこに `.route("エンドポイント", get(ハンドラ))` をどんどん増やしていけばいいよ、という具合だ。

でも、mainを長々と書きたくはない。ここに全てのエンドポイントをまとめるのは嫌だ。どうにか Router を分割して書けないものなのか、というのを 2025 年 03 月くらいに調べていて、あまりうまい検索結果に辿り着けていなかったので、記事にしてみた (遅い: 現在 2025 年 09 月) 。

## 公式ドキュメントに書いてありますよね？
[はい、書いてあります。すみませんでした。](https://docs.rs/axum/latest/axum/struct.Router.html#nesting-routers-with-state)

だがそれに辿り着かないのが人間というもので (主語でかすぎ) 。最終的には公式ドキュメントを見て実装したものの、なんで紹介してる記事になかなか行き着かないのかなーって思っていた。ただ検索が下手っていうだけかもしんないけどさ。

## nest を使って Router を分ける

merge というのもあるようだけど、今回使ったのは nest のほう。
作りたいエンドポイントが、たとえば以下のようなものたちで、`/user` の下は分けて実装したかったら。
```text
GET /
GET /user/
GET /user/{id}/
```

main.rs は、以下みたいになる。
```Rust: main.rs
mod handler;

use axum::{
    http::StatusCode,
    response::IntoResponse,
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    let router = Router::new()
        .route("/", get(root))
        .nest("/user", handler::user::build_router());

    let listener
        = tokio::net::TcpListener::bind("0.0.0.0:8888").await.unwrap();
    axum::serve(listener, router).await.unwrap();
}

async fn root() -> impl IntoResponse {
    (StatusCode::OK, "hello!".to_string())
}
```

main.rs と同じディレクトリに、handler.rs ファイルと handler ディレクトリを作って、handler.rs はモジュールを参照するだけ。

```Rust: handler.rs
pub mod user;
```

handler ディレクトリ内に user.rs ファイルを作って、その中に main.rs で nest の引数の場所で呼んでいる `handler::user::build_router()` 関数 (`pub(crate) fn build_router() -> Router<()> {`) を作成する。

```Rust: handler/user.rs
use axum::{
    extract::{
        Path,
    },
    http::StatusCode,
    response::IntoResponse,
    routing::get,
    Router,
};

pub(crate) fn build_router() -> Router<()> {
    Router::new()
    .route("/", get(get_all_user))
    .route("/{id}/", get(get_user))
}

async fn get_all_user() -> impl IntoResponse {
    (StatusCode::OK, "all_user".to_string())
}

async fn get_user(
    Path(id): Path<(i32)>,
) -> impl IntoResponse {
    (StatusCode::OK, "user".to_string())
}
```

ファイルツリーは以下みたいな状態になっている。
```text
src/main.rs
src/handler.rs
src/handler/user.rs
```

:::message
とりあえず handler ディレクトリに入れたけれど、handler ディレクトリも handler.rs も作らずに main.rs と user.rs だけでもモジュールの参照を調節したらできるはず。
:::

エンドポイントとしては、
```text: 全体
GET /
GET /user/
GET /usre/{id}/
```
を、
```text: main.rs 内
GET /
nest /user handler::user::build_router()
```
と、`/user` 以下に分けて書ける、というわけだ。
```text: user.rs 内
GET /
GET /{id}/
```

これで無事に Router を分けてエンドポイントごとにまとめて書くことができた。

## with_state を使う場合
エンドポイントで処理を開始したら、DB アクセスとかしたいときにコネクションプールとかを受け取りたいわけで、そういうときに with_state を使うことになる。システムワイドで使いたいものを入れておく場所が state なんだと思う。
Router を分けたときには、このあたりは以下みたいに書くことになる。

まず、main.rs にだけ with_state は書く。(以降のコードは説明に必要な部分のみ抜粋)
```Rust: main.rs
use std::sync::Arc;

struct AppState {
    db: Db,
}

#[tokio::main]
async fn main() {
    let app_state = Arc::new(AppState {
        db: 適宜コネクションプールとか,
    });

    let router = Router::new()
        .route("/", get(root))
        .nest("/user", handler::user::build_router())
        .with_state(app_state);

    let listener
        = tokio::net::TcpListener::bind("0.0.0.0:8888").await.unwrap();
    axum::serve(listener, router).await.unwrap();
}
```

`std::sync::Arc` を使って、非同期のエンドポイント間でも受け渡せるようにした構造体 AppState を用意し、初期化して with_state に渡す。
分けたほうの user.rs にある Router では build_router の返すものを `Router<Arc<AppState>>` に変更する。
こっちでは with_state はしない。

```Rust: handler/user.rs
use std::sync::Arc;

use axum::{
    extract::{
        State,
    },
};

use crate::AppState;

pub(crate) fn build_router() -> Router<Arc<AppState>> {
    Router::new()
    .route("/", get(get_all_user))
    .route("/{id}/", get(get_user))
}

async fn get_user(
    Path(id): Path<(i32)>,
    State(state): State<Arc<AppState>>,
) -> impl IntoResponse {
    (StatusCode::OK, "user".to_string())
}
```

エンドポイントのハンドラ get_user のほうでは、引数で state を受け取るようにして、この中のコネクションプールとかを使って DB アクセスする。

## この記事の axum のバージョン
`axum = "0.8.1"` を使用しています。
最後に書くのは不親切ではあるけど、なんとなく最後になっちゃった。


