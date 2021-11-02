<!--
# The alloc crate
-->
# allocクレート

<!--
Initially added: ![Minimum Rust version: 1.36](https://img.shields.io/badge/Minimum%20Rust%20Version-1.36-brightgreen.svg)
-->
![Minimum Rust version: 1.36](https://img.shields.io/badge/Minimum%20Rust%20Version-1.36-brightgreen.svg)で最初に導入されました。

<!--
Before 1.36.0, the standard library consisted of the crates `std`, `core`, and `proc_macro`.
The `core` crate provided core functionality such as `Iterator` and `Copy`
and could be used in `#![no_std]` environments since it did not impose any requirements.
Meanwhile, the `std` crate provided types like `Box<T>` and OS functionality
but required a global allocator and other OS capabilities in return.
-->
1.36.0以前は、 標準ライブラリは`std`、`core`、`proc_macro`クレートで構成されていました。
`core`クレートは`Iterator`や`Copy`のような主要機能を提供し、何も前提条件がなかったので`#![no_std]`環境で使うことができました。
一方、`std`クレートは`Box<T>`のような型やOSの機能を提供していましたが、その代わりにグローバルアロケータやその他のOSの機能を必要としていました。

<!--
Starting with Rust 1.36.0, the parts of `std` that depend on a global allocator, e.g. `Vec<T>`,
are now available in the `alloc` crate. The `std` crate then re-exports these parts.
While `#![no_std]` *binaries* using `alloc` still require nightly Rust,
`#![no_std]` *library* crates can use the `alloc` crate in stable Rust.
Meanwhile, normal binaries, without `#![no_std]`, can depend on such library crates.
We hope this will facilitate the development of a `#![no_std]` compatible ecosystem of libraries
prior to stabilizing support for `#![no_std]` binaries using `alloc`.
-->

Rust 1.36.0,から、`std`の中で、`Vec<T>`のようなグローバルアロケータに依存している部分は`alloc`クレートで提供されるようになりました。
`std`クレートはこれらを再公開します。
`alloc`を使う`#![no_std]`の*バイナリ*はまだnightly版のRustを必要としますが、`#![no_std]`*ライブラリ*クレートは安定版のRustで`alloc`クレートを使うことができます。
一方で、`#![no_std]`ではない通常のバイナリはそのようなライブラリクレートに依存することが可能です。
これにより、`alloc`を使う`#![no_std]`バイナリのサポートが安定化する前に、`#![no_std]`互換のライブラリエコシステムの開発が進むことを期待しています。

<!--
If you are the maintainer of a library that only relies on some allocation primitives to function,
consider making your library `#[no_std]` compatible by using the following at the top of your `lib.rs` file:
-->
もしあなたが幾つかのアロケーションプリミティブのみに依存するライブラリのメンテナだったら、以下をあなたの`lib.rs`の先頭に入れて、`#[no_std]`互換にすることを検討してみてください。

```rust,ignore
#![no_std]

extern crate alloc;

use alloc::vec::Vec;
```
