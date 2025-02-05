<!-- 
# Changes to the prelude 
-->

# Prelude への変更点

<!-- 
## Summary 
-->

## 概要

<!-- 
- The [`Future`] and [`IntoFuture`] traits are now part of the prelude.
- This might make calls to trait methods ambiguous which could make some code fail to compile. 
-->

- [`Future`] と [`IntoFuture`] トレイトが Prelude に含まれるようになりました。
- その結果、一部のコードでトレイトメソッドの呼び出しがあいまいになり、コンパイルに失敗する可能性があります。

[`Future`]: ../../std/future/trait.Future.html
[`IntoFuture`]: ../../std/future/trait.IntoFuture.html

<!-- 
## Details 
-->

## 詳細

<!-- 
The [prelude of the standard library](../../std/prelude/index.html) is the module containing everything that is automatically imported in every module.
It contains commonly used items such as `Option`, `Vec`, `drop`, and `Clone`. 
-->

[標準ライブラリのPrelude](../../std/prelude/index.html) は、すべてのモジュールで自動的にインポートされる項目をまとめたモジュールです。例えば、`Option`、`Vec`、`drop`、`Clone` など、よく使われる項目が含まれています。


<!-- 
The Rust compiler prioritizes any manually imported items over those from the prelude,
to make sure additions to the prelude will not break any existing code.
For example, if you have a crate or module called `example` containing a `pub struct Option;`,
then `use example::*;` will make `Option` unambiguously refer to the one from `example`;
not the one from the standard library. 
-->

Rust コンパイラは、Prelude からインポートされる項目よりも、明示的にインポートされた項目を優先します。たとえば、`example` というクレートやモジュールに `pub struct Option;` が定義されている場合、`use example::*;` とすれば、`Option` は標準ライブラリのものではなく `example` のものが使われます。

<!-- 
However, adding a _trait_ to the prelude can break existing code in a subtle way.
For example, a call to `x.poll()` which comes from a `MyPoller` trait might fail to compile if `std`'s `Future` is also imported, because the call to `poll` is now ambiguous and could come from either trait. 
-->

ただし、_トレイト_ を Prelude に追加すると、既存のコードに微妙な影響が生じることがあります。たとえば、`MyPoller` というトレイトの `poll` メソッドを呼び出しているコードに、同時に標準ライブラリの `Future` トレイトもインポートされている場合、どちらの `poll` を呼ぶべきかがあいまいになり、コンパイルに失敗する可能性があります。

<!-- 
As a solution, Rust 2024 will use a new prelude.
It's identical to the current one, except for the following changes: 
-->

そのため、Rust 2024 では新しい Prelude が導入されます。基本的な内容は現行のものと変わりませんが、以下の項目が追加されています:

<!-- 
- Added: 
-->
- 追加された項目：
    - [`std::future::Future`][`Future`]
    - [`std::future::IntoFuture`][`IntoFuture`]

<!-- 
## Migration 
-->

## 移行

<!-- 
### Conflicting trait methods 
-->

### 競合するトレイトメソッド

<!-- 
When two traits that are in scope have the same method name, it is ambiguous which trait method should be used. For example: 
-->

スコープ内にある二つのトレイトが同じメソッド名を持つ場合、どちらのトレイトメソッドを使うべきかがあいまいになってしまいます。例えば、次のコードをご覧ください。

```rust,edition2021
trait MyPoller {
    // This name is the same as the `poll` method on the `Future` trait from `std`.
    fn poll(&self) {
        println!("polling");
    }
}

impl<T> MyPoller for T {}

fn main() {
    // Pin<&mut async {}> implements both `std::future::Future` and `MyPoller`.
    // If both traits are in scope (as would be the case in Rust 2024),
    // then it becomes ambiguous which `poll` method to call
    core::pin::pin!(async {}).poll();
}
```

<!-- 
We can fix this so that it works on all editions by using fully qualified syntax: 
-->

これをすべてのエディションで正しく動作させるためには、完全修飾構文を使う必要があります。

```rust,ignore
fn main() {
    // Now it is clear which trait method we're referring to
    <_ as MyPoller>::poll(&core::pin::pin!(async {}));
}
```

<!-- 
The [`rust_2024_prelude_collisions`] lint will automatically modify any ambiguous method calls to use fully qualified syntax. This lint is part of the `rust-2024-compatibility` lint group, which will automatically be applied when running `cargo fix --edition`. To migrate your code to be Rust 2024 Edition compatible, run: 
-->

また、[`rust_2024_prelude_collisions`] リントは、あいまいなメソッド呼び出しを自動的に完全修飾構文に変更してくれます。このリントは `rust-2024-compatibility` リントグループの一部であり、`cargo fix --edition` を実行すると自動的に適用されます。Rust 2024 エディションに互換性のあるコードに移行するには、次のコマンドを実行してください。

```sh
cargo fix --edition
```

<!-- 
Alternatively, you can manually enable the lint to find places where these qualifications need to be added: 
-->

もしくは、完全修飾が必要な箇所を見つけるために、リントを手動で有効にすることもできます。

```rust
// Add this to the root of your crate to do a manual migration.
#![warn(rust_2024_prelude_collisions)]
```

[`rust_2024_prelude_collisions`]: ../../rustc/lints/listing/allowed-by-default.html#rust-2024-prelude-collisions
