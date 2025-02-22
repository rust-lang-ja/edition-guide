<!--
# Unsafe `extern` blocks
-->

# アンセーフな `extern` ブロック

<!--
## Summary
-->

## 概要

<!--
- [`extern` blocks] must now be marked with the `unsafe` keyword.
-->

- [`extern` ブロック]は `unsafe` として宣言される必要があります。

<!--
[`extern` blocks]: ../../reference/items/external-blocks.html
-->

[`extern` ブロック]: https://doc.rust-lang.org/reference/items/external-blocks.html

<!--
## Details
-->

## 詳細

<!--
Rust 1.82 added the ability in all editions to mark [`extern` blocks] with the `unsafe` keyword.[^RFC3484] Adding the `unsafe` keyword helps to emphasize that it is the responsibility of the author of the `extern` block to ensure that the signatures are correct. If the signatures are not correct, then it may result in undefined behavior.
-->

Rust 1.82 から、全エディションで [`extern` ブロック]を `unsafe` キーワードとともに宣言できるようになりました。[^RFC3484]
`unsafe` と書くことで、`extern` を書いた人にシグネチャが正しいことを保証する責任があることが明確になります。
シグネチャが正しくないと、未定義動作を引き起こしかねません。

<!--
The syntax for an unsafe `extern` block looks like this:
-->

以下に、アンセーフな `extern` ブロック構文の例を示します。

<!--
```rust
unsafe extern "C" {
    // sqrt (from libm) may be called with any `f64`
    pub safe fn sqrt(x: f64) -> f64;

    // strlen (from libc) requires a valid pointer,
    // so we mark it as being an unsafe fn
    pub unsafe fn strlen(p: *const std::ffi::c_char) -> usize;

    // this function doesn't say safe or unsafe, so it defaults to unsafe
    pub fn free(p: *mut core::ffi::c_void);

    pub safe static IMPORTANT_BYTES: [u8; 256];
}
```
-->

```rust
unsafe extern "C" {
    // (libm の) sqrt 関数は任意の `f64` 引数を取れる
    pub safe fn sqrt(x: f64) -> f64;

    // (libc の) strlen 関数は有効なポインタを要求するため、
    // アンセーフな関数として宣言する
    pub unsafe fn strlen(p: *const std::ffi::c_char) -> usize;

    // safe も unsafe もない関数はデフォルトではアンセーフとなる
    pub fn free(p: *mut core::ffi::c_void);

    pub safe static IMPORTANT_BYTES: [u8; 256];
}
```

<!--
In addition to being able to mark an `extern` block as `unsafe`, you can also specify if individual items in the `extern` block are `safe` or `unsafe`. Items marked as `safe` can be used without an `unsafe` block.
-->

`extern` ブロックを `unsafe` とできるのに加え、`extern` ブロック内の個々のアイテムを `safe` か `unsafe` と明示できます。
`safe` のついたアイテムは、`unsafe` で囲まなくても使用できるようになります。

<!--
Starting with the 2024 Edition, it is now required to include the `unsafe` keyword on an `extern` block. This is intended to make it very clear that there are safety requirements that must be upheld by the extern definitions.
-->

2024 エディション以降、`extern` ブロックには必ず `unsafe` を付与する必要があります。
これにより、extern 定義が遵守すべき安全性要件が存在するということが明確になることが期待されます。

<!--
[^RFC3484]: See [RFC 3484](https://github.com/rust-lang/rfcs/blob/master/text/3484-unsafe-extern-blocks.md) for the original proposal.
-->

[^RFC3484]: 元の提案については [RFC 3484](https://github.com/rust-lang/rfcs/blob/master/text/3484-unsafe-extern-blocks.md) を参照のこと。

<!--
## Migration
-->

## 移行

<!--
The [`missing_unsafe_on_extern`] lint can update `extern` blocks to add the `unsafe` keyword. The lint is part of the `rust-2024-compatibility` lint group which is included in the automatic edition migration. In order to migrate your code to be Rust 2024 Edition compatible, run:
-->

[`missing_unsafe_on_extern`] リントで `extern` ブロックに `unsafe` を自動付与できます。
このリントは、自動エディション移行に含まれる `rust-2024-compatibility` リントグループの一部です。
コードを Rust 2024 互換に移行するには、以下を実行します。

```sh
cargo fix --edition
```

<!--
Just beware that this automatic migration will not be able to verify that the signatures in the `extern` block are correct. It is still your responsibility to manually review their definition.
-->

ただし、自動移行ツールは、`extern` ブロックのシグネチャが正しいことを検証することはできません。
それを保証するのは、変わらずプログラマの責任です。

<!--
Alternatively, you can manually enable the lint to find places where there are `unsafe` blocks that need to be updated.
-->

変更の必要がある `unsafe` ブロックを検出するリントを手動で有効化することもできます。

<!--
```rust
// Add this to the root of your crate to do a manual migration.
#![warn(missing_unsafe_on_extern)]
```
-->

```rust
// クレートのトップレベルに以下を追加すると手動移行できる
#![warn(missing_unsafe_on_extern)]
```

<!--
[`missing_unsafe_on_extern`]: ../../rustc/lints/listing/allowed-by-default.html#missing-unsafe-on-extern
-->

[`missing_unsafe_on_extern`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#missing-unsafe-on-extern
