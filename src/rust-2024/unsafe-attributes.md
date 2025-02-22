<!--
# Unsafe attributes
-->

# アンセーフなアトリビュート

<!--
## Summary
-->

## 概要

<!--
- The following attributes must now be marked as `unsafe`:
-->
- 以下のアトリビュートに `unsafe` が必要になりました。
    - [`export_name`]
    - [`link_section`]
    - [`no_mangle`]

<!--
[`export_name`]: ../../reference/abi.html#the-export_name-attribute
[`link_section`]: ../../reference/abi.html#the-link_section-attribute
[`no_mangle`]: ../../reference/abi.html#the-no_mangle-attribute
-->

[`export_name`]: https://doc.rust-lang.org/reference/abi.html#the-export_name-attribute
[`link_section`]: https://doc.rust-lang.org/reference/abi.html#the-link_section-attribute
[`no_mangle`]: https://doc.rust-lang.org/reference/abi.html#the-no_mangle-attribute

<!--
## Details
-->

## 詳細

<!--
Rust 1.82 added the ability in all editions to mark certain attributes as `unsafe` to indicate that they have soundness requirements that must be upheld.[^RFC3325] The syntax for an unsafe attribute looks like this:
-->

Rust 1.82 から全エディションにおいて、アトリビュートが健全性要件の遵守を要する場合に `unsafe` と書ける機能が追加されました。[^RFC3325]
アンセーフなアトリビュートは以下のように使用します。

<!--
```rust
// SAFETY: there is no other global function of this name
#[unsafe(no_mangle)]
pub fn example() {}
```
-->

```rust
// SAFETY: 同名のグローバル関数は他に存在しない
#[unsafe(no_mangle)]
pub fn example() {}
```

<!--
Marking the attribute with `unsafe` highlights that there are safety requirements that must be upheld that the compiler cannot verify on its own.
-->

アトリビュートが `unsafe` だと宣言することで、遵守されるべき安全性要件があって、それをコンパイラが保証できないことが明示できます。

<!--
Starting with the 2024 Edition, it is now required to mark these attributes as `unsafe`. The following section describes the safety requirements for these attributes.
-->

2024 エディションからは、上記のアトリビュートを `unsafe` とすることが必須となります。
実際に遵守すべき安全性要件を以下に説明します。

<!--
[^RFC3325]: See [RFC 3325](https://rust-lang.github.io/rfcs/3325-unsafe-attributes.html) for the original proposal.
-->

[^RFC3325]: 元の提案は [RFC 3325](https://rust-lang.github.io/rfcs/3325-unsafe-attributes.html) を参照のこと。

<!--
### Safety requirements
-->

### 安全性要件

<!--
The [`no_mangle`], [`export_name`], and [`link_section`] attributes influence the symbol names and linking behavior of items. Care must be taken to ensure that these attributes are used correctly.
-->

アトリビュート [`no_mangle`]、[`export_name`]、[`link_section`] はシンボル名とリンク動作に関わるため、細心の注意を払って使用しなくてはなりません。

<!--
Because the set of symbols across all linked libraries is a global namespace, there can be issues if there is a symbol name collision between libraries. Typically this isn't an issue for normally defined functions because [symbol mangling] helps ensure that the symbol name is unique. However, attributes like `export_name` can upset that assumption of uniqueness.
-->

リンクされるシンボルの名前空間は全ライブラリで共有なので、ライブラリ間でシンボル名が衝突すると問題が発生する可能性があります。
通常の方法で定義された関数は、[シンボルのマングリング]のおかげで基本的に名前の衝突を気にする必要はありませんが、`export_name` といったアトリビュートを使うとそうはいかなくなります。

<!--
For example, in previous editions the following crashes on most Unix-like platforms despite containing only safe code:
-->

例えば、以下のコードは何もアンセーフと書いていませんが、過去のエディションでは Unix 系の環境でクラッシュします。

```rust,no_run,edition2021
fn main() {
    println!("Hello, world!");
}

#[export_name = "malloc"]
fn foo() -> usize { 1 }
```

<!--
In the 2024 Edition, it is now required to mark these attributes as unsafe to emphasize that it is required to ensure that the symbol is defined correctly:
-->

2024 エディションでは、上記のアトリビュートには `unsafe` をつける必要があり、シンボルが適切に定義される必要があることが明確になります。

<!--
```rust,edition2024
// SAFETY: There should only be a single definition of the loop symbol.
#[unsafe(export_name="loop")]
fn arduino_loop() {
    // ...
}
```
-->

```rust,edition2024
// SAFETY: loop というシンボルの定義は以下のみ
#[unsafe(export_name="loop")]
fn arduino_loop() {
    // ...
}
```

<!--
[symbol mangling]: ../../rustc/symbol-mangling/index.html
[`unsafe_attr_outside_unsafe`]: ../../rustc/lints/listing/allowed-by-default.html#unsafe-attr-outside-unsafe
-->

[シンボルのマングリング]: https://doc.rust-lang.org/rustc/symbol-mangling/index.html
[`unsafe_attr_outside_unsafe`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unsafe-attr-outside-unsafe

<!--
## Migration
-->

## 移行

<!--
The [`unsafe_attr_outside_unsafe`] lint can update these attributes to use the `unsafe(...)` format. The lint is part of the `rust-2024-compatibility` lint group which is included in the automatic edition migration. In order to migrate your code to be Rust 2024 Edition compatible, run:
-->

[`unsafe_attr_outside_unsafe`] リントで、上記のアトリビュートに `unsafe(...)` を自動付与できます。
このリントは、自動エディション移行に含まれる `rust-2024-compatibility` リントグループの一部です。
コードを Rust 2024 互換に移行するには、以下を実行します。

```sh
cargo fix --edition
```

<!--
Just beware that this automatic migration will not be able to verify that these attributes are being used correctly. It is still your responsibility to manually review their usage.
-->

ただし、自動移行ツールはアトリビュートの使用方法が正しいことを検証するわけではありません。
それを保証するのは、変わらずプログラマの責任です。

<!--
Alternatively, you can manually enable the lint to find places where these attributes need to be updated.
-->

変更の必要のあるアトリビュートを検出するリントを手動で有効化することもできます。

<!--
```rust
// Add this to the root of your crate to do a manual migration.
#![warn(unsafe_attr_outside_unsafe)]
```
-->

```rust
// クレートのトップレベルに以下を追加すると手動移行できる
#![warn(unsafe_attr_outside_unsafe)]
```
