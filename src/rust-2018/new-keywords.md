<!--
# New keywords
-->

# 新しいキーワード

<!--
![Minimum Rust version: 1.27](https://img.shields.io/badge/Minimum%20Rust%20Version-1.27-brightgreen.svg)
-->

![導入 Rust バージョン: 1.27](https://img.shields.io/badge/%E5%B0%8E%E5%85%A5%20Rust%20%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3-1.27-brightgreen.svg)

<!--
## Summary
-->

## 概要

<!--
- `dyn` is a [strict keyword][strict], in 2015 it is a [weak keyword].
- `async` and `await` are [strict keywords][strict].
- `try` is a [reserved keyword].
-->

- `dyn` は[正格キーワード][strict]です。2015 では[弱いキーワード]です。
- `asnyc` と `await` は[正格キーワード][strict]です。
- `try` は[予約キーワード]です。

<!--
[strict]: https://doc.rust-lang.org/reference/keywords.html#strict-keywords
[weak keyword]: https://doc.rust-lang.org/reference/keywords.html#weak-keywords
[reserved keyword]: https://doc.rust-lang.org/reference/keywords.html#reserved-keywords
-->

[strict]: https://doc.rust-lang.org/reference/keywords.html#strict-keywords
[弱いキーワード]: https://doc.rust-lang.org/reference/keywords.html#weak-keywords
[予約キーワード]: https://doc.rust-lang.org/reference/keywords.html#reserved-keywords

<!--
## Motivation
-->

## 動機

<!--
### `dyn Trait` for trait objects
-->

### トレイトオブジェクトを表す `dyn Trait`

<!--
The `dyn Trait` feature is the new syntax for using trait objects. In short:
-->

`dyn Trait` 機能は、トレイトオブジェクトを使うための新しい構文です。簡単に言うと:

<!--
* `Box<Trait>` becomes `Box<dyn Trait>`
* `&Trait` and `&mut Trait` become `&dyn Trait` and `&mut dyn Trait`
-->

* `Box<Trait>` は `Box<dyn Trait>` になり、
* `&Trait` と `&mut Trait` は `&dyn Trait` と `&mut dyn Trait` になる

<!--
And so on. In code:
-->

といった具合です。プログラム内では:

```rust
trait Trait {}

impl Trait for i32 {}

// old
// いままで
fn function1() -> Box<Trait> {
# unimplemented!()
}

// new
// これから
fn function2() -> Box<dyn Trait> {
# unimplemented!()
}
```

<!--
That's it!
-->

これだけです！

<!--
#### Why?
-->

#### なぜ？

<!--
Using just the trait name for trait objects turned out to be a bad decision.
The current syntax is often ambiguous and confusing, even to veterans,
and favors a feature that is not more frequently used than its alternatives,
is sometimes slower, and often cannot be used at all when its alternatives can.
-->

トレイトオブジェクトにトレイト名をそのまま使うのは悪手だったと、後になって分かりました。
今までの構文は、経験者にとってさえ往々にして曖昧にして難解で、代替機能を使うべきで本来お呼びでないような場面[^1]で頻繁に使われ、時には遅く、代替機能にはできることができないのです。

<!--
Furthermore, with `impl Trait` arriving, "`impl Trait` vs `dyn Trait`" is much
more symmetric, and therefore a bit nicer, than "`impl Trait` vs `Trait`".
`impl Trait` is explained [here][impl-trait].
-->

その上、`impl Trait` が入ったことで、「`impl Trait` か `dyn Trait` か」の関係はより対称的になり、「`impl Trait` か `Trait` か」よりちょっといい感じです。
`impl Trait` の説明は[こちら][impl-trait]です。

<!--
In the new edition, you should therefore prefer `dyn Trait` to just `Trait`
where you need a trait object.
-->

したがって、新しいエディションでは、トレイトオブジェクトが必要なときは、ただの `Trait` でなく `dyn Trait` を使うべきです。

<!--
[impl-trait]: ../../rust-by-example/trait/impl_trait.html
-->
[impl-trait]: https://doc.rust-jp.rs/rust-by-example-ja/trait/impl_trait.html


> [^1] *訳注*:
原文ではこの文は、本ページで説明する新構文を提案する [RFC](https://rust-lang.github.io/rfcs/2113-dyn-trait-syntax.html) から抜粋された文章になっています。
特に脚注で示した箇所は、原文では "favors a feature that is not more frequently used than its alternatives" とあり、その文意は[同 RFC に解説されています](https://rust-lang.github.io/rfcs/2113-dyn-trait-syntax.html#favors-a-feature-that-is-not-more-frequently-used-than-its-alternatives)。以下では、それを要約します。  
特定のトレイトを実装した異なる型を共通して扱いたいとき、大抵はトレイトオブジェクトを使う必要はありません。
単一のコンテナに複数の型の構造体を入れたい場合、`enum` を使えばよいです。
関数の返り値が特定のトレイトを実装していると示すには、`impl Trait` 構文を使えばよいです。
特定のトレイトを実装する任意の型を関数の引数や構造体のフィールドにした場合、ジェネリクスを使えばよいです。
大抵の場合は、このようにセマンティクス面からもパフォーマンス面からもより適切な代替案があり、トレイトオブジェクトの出る幕はありません。  
トレイトオブジェクトが真に必要なのは、これより複雑なことをしたい場合だけです。
しかし、Rust 2015 では、`&Trait` のように書くだけで、「気軽に」トレイトオブジェクトが作れてしまうという罠がありました。
そこで、Rust 2018 では、どうしてもトレイトオブジェクトを作りたい場合は `&dyn Trait` 構文を使用することが必要になりました。

<!--
### `async` and `await`
-->

### `async` と `await`

<!--
These keywords are reserved to implement the async-await feature of Rust, which was ultimately [released to stable in 1.39.0](https://blog.rust-lang.org/2019/11/07/Async-await-stable.html).
-->

これらのキーワードは Rust に非同期の機能を実装するために予約されました。非同期の機能は[最終的に 1.39.0 でリリースされました](https://blog.rust-lang.org/2019/11/07/Async-await-stable.html)。

<!--
### `try` keyword
-->

### キーワード `try`

<!--
The `try` keyword is reserved for use in `try` blocks, which have not (as of this writing) been stabilized ([tracking issue](https://github.com/rust-lang/rust/issues/31436))
-->

キーワード `try` は `try` ブロックで使うために予約されましたが、（これを書いている時点で）まだ安定化されていません（[追跡イシュー](https://github.com/rust-lang/rust/issues/31436)）
