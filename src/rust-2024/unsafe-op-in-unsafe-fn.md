<!--
# unsafe_op_in_unsafe_fn warning
-->

# unsafe_op_in_unsafe_fn 警告

<!--
## Summary
-->

## 概要

<!--
- The [`unsafe_op_in_unsafe_fn`] lint now warns by default.
  This warning detects calls to unsafe operations in unsafe functions without an explicit unsafe block.
-->

- [`unsafe_op_in_unsafe_fn`] リントのデフォルトのレベルが警告になりました。
  アンセーフな関数中でアンセーフな操作を明示的な unsafe ブロックなしに呼び出した場合にこの警告が出ます。

<!--
[`unsafe_op_in_unsafe_fn`]: ../../rustc/lints/listing/allowed-by-default.html#unsafe-op-in-unsafe-fn
-->

[`unsafe_op_in_unsafe_fn`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unsafe-op-in-unsafe-fn

<!--
## Details
-->

## 詳細

<!--
The [`unsafe_op_in_unsafe_fn`] lint will fire if there are [unsafe operations] in an unsafe function without an explicit [`unsafe {}` block][unsafe-block].
-->

[`unsafe_op_in_unsafe_fn`] は、アンセーフな関数中に[アンセーフな操作]が行われているが、それが [`unsafe {}` ブロック][unsafe-block]で囲まれていないことを検出するリントです。

```rust
# #![warn(unsafe_op_in_unsafe_fn)]
unsafe fn get_unchecked<T>(x: &[T], i: usize) -> &T {
  x.get_unchecked(i) // WARNING: requires unsafe block
                     // (訳) 警告: unsafe ブロックが必要
}
```

<!--
The solution is to wrap any unsafe operations in an `unsafe` block:
-->

これに対処するには、アンセーフな操作をすべて `unsafe` ブロックで囲ってください。

```rust
# #![deny(unsafe_op_in_unsafe_fn)]
unsafe fn get_unchecked<T>(x: &[T], i: usize) -> &T {
  unsafe { x.get_unchecked(i) }
}
```

<!--
This change is intended to help protect against accidental use of unsafe operations in an unsafe function.
The `unsafe` function keyword was performing two roles.
One was to declare that *calling* the function requires unsafe, and that the caller is responsible to uphold additional safety requirements.
The other role was to allow the use of unsafe operations inside of the function.
This second role was determined to be too risky without explicit `unsafe` blocks.
-->

これは、アンセーフ関数内の意図しないアンセーフな操作を避けるためのものです。
かつては、アンセーフ関数の `unsafe` というキーワードには二重の意味が込められていました。
関数を**呼び出す**操作がアンセーフであり、呼び出し側が満たすべき安全性要件が存在するという点と、関数内でアンセーフな操作ができるという点です。
特に後者を `unsafe` ブロックなしで実行できることが危険すぎると判断されました。

<!--
More information and motivation may be found in [RFC #2585].
-->

詳細情報や動機などは [RFC #2585] に詳しいです。

<!--
[unsafe operations]: ../../reference/unsafety.html
[unsafe-block]: ../../reference/expressions/block-expr.html#unsafe-blocks
[RFC #2585]: https://rust-lang.github.io/rfcs/2585-unsafe-block-in-unsafe-fn.html
-->

[アンセーフな操作]: https://doc.rust-lang.org/reference/unsafety.html
[unsafe-block]: https://doc.rust-lang.org/reference/expressions/block-expr.html#unsafe-blocks
[RFC #2585]: https://rust-lang.github.io/rfcs/2585-unsafe-block-in-unsafe-fn.html

<!--
## Migration
-->

## 移行

<!--
The [`unsafe_op_in_unsafe_fn`] lint is part of the `rust-2024-compatibility` lint group.
In order to migrate your code to be Rust 2024 Edition compatible, run:
-->

[`unsafe_op_in_unsafe_fn`] リントは `rust-2024-compatibility` リントグループの一部です。
以下を実行すると、コードを Rust 2024 互換に変換できます。

```sh
cargo fix --edition
```

<!--
Alternatively, you can manually enable the lint to find places where unsafe blocks need to be added, or switch it to `allow` to silence the lint completely.
-->

あるいは、unsafe ブロックを追加すべき場所を見つけるために以下のように手動でリントを有効化したり、逆に `allow` としてリントを完全に無視することもできます。

<!--
```rust
// Add this to the root of your crate to do a manual migration.
#![warn(unsafe_op_in_unsafe_fn)]
```
-->

```rust
// クレートのトップレベルに以下を追加すると手動移行できる
#![warn(unsafe_op_in_unsafe_fn)]
```
