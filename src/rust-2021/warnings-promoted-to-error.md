<!--
# Warnings promoted to errors
-->

# 警告からエラーへの格上げ

<!--
## Summary
-->

## 概要

<!--
- Code that triggered the `bare_trait_objects` and `ellipsis_inclusive_range_patterns` lints will error in Rust 2021.
-->

- `bare_trait_objects` リントか `ellipsis_inclusive_range_patterns` リントが出るコードは、Rust 2021 ではエラーになります。

<!--
## Details
-->

## 詳細

<!--
Two existing lints are becoming hard errors in Rust 2021, but these lints will remain warnings in older editions.
-->

現存する2つのリントが Rust 2021 ではエラーになります。古いエディションではこれらは警告のままです。

### `bare_trait_objects`:

<!--
The use of the `dyn` keyword to identify [trait objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html)
will be mandatory in Rust 2021.
-->

Rust 2021 では、[トレイトオブジェクト](https://doc.rust-jp.rs/book-ja/ch17-02-trait-objects.html)を表すために、`dyn` キーワードを使用することが必須になりました。

<!--
For example, the following code which does not include the `dyn` keyword in `&MyTrait`
will produce an error instead of just a lint in Rust 2021:
-->

例えば、以下のコードでは `&MyTrait` に `dyn` キーワードが含まれていないため、Rust 2021 ではただのリントではなくエラーが発生します:

```rust
pub trait MyTrait {}

pub fn my_function(_trait_object: &MyTrait) { // should be `&dyn MyTrait`
                                              // `&dyn MyTrait` と書かなくてはならない
  unimplemented!()
}
```

### `ellipsis_inclusive_range_patterns`:

<!--
The [deprecated `...` syntax](https://doc.rust-lang.org/stable/reference/patterns.html#range-patterns)
for inclusive range patterns (i.e., ranges where the end value is *included* in the range) is no longer 
accepted in Rust 2021. It has been superseded by `..=`, which is consistent with expressions.
-->

閉区間パターン（つまり、終端の値を*含む*範囲）を表す、[非推奨の `...` 構文](https://doc.rust-lang.org/stable/reference/patterns.html#range-patterns)は、Rust 2021 では使えなくなります。
式との統一性のため、`..=` を使うことが推奨されていました。

<!--
For example, the following code which uses `...` in a pattern will produce an error instead of 
just a lint in Rust 2021:
-->

例えば、次のコードはパターンとして `...` を使っているため、Rust 2021 ではただのリントではなくエラーが発生します:

```rust
pub fn less_or_eq_to_100(n: u8) -> bool {
  matches!(n, 0...100) // should be `0..=100`
                       // `0..=100` と書かなくてはならない
}
```

<!--
## Migrations 
-->

## 移行

<!--
If your Rust 2015 or 2018 code does not produce any warnings for `bare_trait_objects` 
or `ellipsis_inclusive_range_patterns` and you've not allowed these lints through the 
use of `#![allow()]` or some other mechanism, then there's no need to migrate.
-->

あなたの Rust 2015 か 2018 のコードで、`bare_trait_objects` や `ellipsis_inclusive_range_patterns` といったエラーが出ず、`#![allow()]` などを使ってこれらのリントを許可する設定をしていなければ、移行の際にやることはありません。

<!--
To automatically migrate any crate that uses `...` in patterns or does not use `dyn` with
trait objects, you can run `cargo fix --edition`.
-->

`...` をパターンに使用していたり、トレイトオブジェクトに `dyn` を使っていないクレートがある場合は、
`cargo fix --edition` を実行すればよいです。
