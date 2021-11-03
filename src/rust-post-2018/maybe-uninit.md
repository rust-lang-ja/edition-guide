# MaybeUninit

<!--
Initially added: ![Minimum Rust version: 1.36](https://img.shields.io/badge/Minimum%20Rust%20Version-1.36-brightgreen.svg)
-->
![Minimum Rust version: 1.36](https://img.shields.io/badge/Minimum%20Rust%20Version-1.36-brightgreen.svg)で最初に導入されました。

<!--
In previous releases of Rust, the [`mem::uninitialized`] function has allowed
you to bypass Rust's initialization checks by pretending that you've
initialized a value at type `T` without doing anything. One of the main uses
of this function has been to lazily allocate arrays.
-->
以前のリリースのRustでは、[`mem::uninitialized`]関数により、何もせずに型Tの値を初期化したように見せかけて、Rustの初期化チェックをバイパスすることができました。
この関数の主要な使い道の一つは配列の遅延アロケートでした。

<!--
However, [`mem::uninitialized`] is an incredibly dangerous operation that
essentially cannot be used correctly as the Rust compiler assumes that values
are properly initialized. For example, calling `mem::uninitialized::<bool>()`
causes *instantaneous __undefined behavior__* as, from Rust's point of view,
the uninitialized bits are neither `0` (for `false`) nor `1` (for `true`) -
the only two allowed bit patterns for `bool`.
-->
しかし、Rustコンパイラは値が正しく初期化されることを想定しているので、[`mem::uninitialized`]関数は、本質的に正しく使うことができない、とても危険な操作です。
例えば、`mem::uninitialized::<bool>()`の呼び出しは*即時の__未定義動作__* を引き起こします。
なぜならば、Rustの視点からは、初期化されていないビットは`bool`の取り得る二つの値である、`0` (`false`) でも `1` (`true`) でもないからです。

<!--
To remedy this situation, in Rust 1.36.0, the type [`MaybeUninit<T>`] has
been stabilized. The Rust compiler will understand that it should not assume
that a [`MaybeUninit<T>`] is a properly initialized `T`. Therefore, you can
do gradual initialization more safely and eventually use `.assume_init()`
once you are certain that `maybe_t: MaybeUninit<T>` contains an initialized
`T`.
-->
この状況を是正するために、Rust 1.36.0で[`MaybeUninit<T>`]型が安定化されました。
Rustコンパイラは、[`MaybeUninit<T>`]は`T`が正しく初期化されると仮定できないことを理解しています。
したがって、段階的な初期化をより安全に行うことができ、`maybe_t: MaybeUninit<T>`が初期化された`T`を含んでいることが確実になったら、 `.assume_init()`を呼び出します。

<!--
As [`MaybeUninit<T>`] is the safer alternative, starting with Rust 1.39, the
function [`mem::uninitialized`] will be deprecated.
-->
[`MaybeUninit<T>`]はより安全な代替手段であるため、Rust 1.39から、[`mem::uninitialized`]関数は非推奨となります。

[`MaybeUninit<T>`]: https://doc.rust-lang.org/std/mem/union.MaybeUninit.html
[`mem::uninitialized`]: https://doc.rust-lang.org/std/mem/fn.uninitialized.html
