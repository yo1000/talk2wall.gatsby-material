---
title: "Rustでコレクション操作"
date: "2019-01-17"
cover: "cover-tech.jpg"
category: "Tech"
tags:
- rust
---

Rustでコレクション操作をしてみようとしたところ、基本的な部分ではありつつも、けっこう躓いたのでメモ。公式リファレンスが充実しているので、そちらを確認するのも良いですが、型引数が必要な箇所や、実際にそれをどう指定するのか等までは、例がまとまっていなかったので。

HTTPリクエストで与えられたクエリ文字列を、JSONに変換してレスポンスする、というのを例にします。

今回のコードサンプルは以下。<br>
https://github.com/yo1000/rust-hyper/tree/4223e61ff6


## 要件
- Rust 1.31.1
- Cargo 1.31.0

```bash
$ rustc -V
rustc 1.31.1 (b6c32da9b 2018-12-18)

$ cargo -V
cargo 1.31.0 (339d9f9c8 2018-11-16)
```


## コード例
今回はコード例から。コード内で使っている操作を個別に補足説明していくスタイル。

```rust{numberLines:true}
(&Method::GET, "/query_as_json") => {
    let query_as_map = match _req.uri().query() {
        Some(it) => {
            it.split('&')
                .map(|q| q.split('=')
                    .collect::<Vec<_>>())
                .filter(|q| q.len() >= 1)
                .map(|q| match q.len() {
                    1 => { (q[0], "") }
                    _ => { (q[0], q[1]) }
                })
                .collect::<HashMap<_, _>>()
        }
        None => { HashMap::new() }
    };

    Response::builder()
        .status(StatusCode::OK)
        .header("Content-Type", "application/json; charset=utf-8")
        .body(Body::from(json!(query_as_map).to_string()))
        .unwrap()
}
```


### split
`split`は多くの他言語と同じく、与えたキーワードで文字列を分割するというものです。`split`メソッドでは、[`Split`](https://doc.rust-lang.org/std/str/struct.Split.html)トレイトが返却されます。`Split`トレイトには、[`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html)トレイトが実装されているため、これを基点に各種コレクション操作メソッドを呼び出せます。


### collect
`collect`も多くの他言語と同じく、操作している反復子を目的のオブジェクトに集めることができです。集める先のオブジェクト型の指定には、`::<_>`でメソッドに型引数を与えたり、代入先の変数に型を明示することで推論させるような方法があります。

今回の例では`collect::<Vec<_>>()`と指定しているので、`Iterator`の各オブジェクトが、`Vec`オブジェクトに集められます。


### map
`map`も多くの他言語と同じく、操作している反復子中の各オブジェクトを加工変換して、変換後のオブジェクトを割り当てるができます。`map`メソッドでは、[`Map`](https://doc.rust-lang.org/std/iter/struct.Map.html)トレイトが返却されます。`Map`トレイトには、`Iterator`トレイトが実装されているため、各種コレクション操作メソッドをチェーンできます。

先の例では、`map(|q| q.split('=').collect::<Vec<_>>())`のように指定しているので、反復子中の各文字列を`=`で分割して、これを`Vec`に集めたものに変換、割り当てをしています。


### filter
`filter`も多くの他言語と同じく、操作している反復子中の各オブジェクトを、条件によって絞り込むことができます。`filter`メソッドでは、[`Filter`](https://doc.rust-lang.org/std/iter/struct.Filter.html)トレイトが返却されます。`Filter`トレイトには、`Iterator`トレイトが実装されているため、各種コレクション操作メソッドをチェーンできます。

先の例では、`filter(|q| q.len() >= 1)`のように指定しているので、反復子中の各行列の長さが1以上のもののみに絞り込みをしています。


## 参考
コレクション操作メソッドは他にもたくさんありますが、基本的に他の言語で使用しているものと機能は同じです。より詳しい内容については、公式ドキュメントの[`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html)トレイトを確認してみてください。

- https://doc.rust-lang.org/std/iter/trait.Iterator.html
- https://qiita.com/lo48576/items/34887794c146042aebf1