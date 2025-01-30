<!--
# Raw lifetimes
-->
# 生ライフタイム

<!--
## Summary
-->

## 概要

<!--
- `'r#ident_or_keyword` is now allowed as a lifetime, which allows using keywords such as `'r#fn`.
-->

- `'r#ident_or_keyword` がライフタイムを表す構文として認められ、`'r#fn` のようにキーワードが使用できるようになりました。

<!--
## Details
-->

## 詳細

<!--
Raw lifetimes are introduced in the 2021 edition to support the ability to migrate to newer editions that introduce new keywords. This is analogous to [raw identifiers] which provide the same functionality for identifiers. For example, the 2024 edition introduced the `gen` keyword. Since lifetimes cannot be keywords, this would cause code that use a lifetime `'gen` to fail to compile. Raw lifetimes allow the migration lint to modify those lifetimes to `'r#gen` which do allow keywords.
-->

生ライフタイムは、新エディションでキーワードが追加された場合の移行を考慮し、Rust 2021 で導入されました。
[生識別子]は識別子に対する機能でしたが、それと同等です。
たとえば、2024 エディションでは `gen` キーワードが導入されました。
しかし、ライフタイム名にキーワードは使えないことから、これではライフタイム `'gen` が使われたコードがコンパイルできなくなってしまいます。
生ライフタイムにはキーワードも使えるので、移行リントは `'gen` を `'r#gen` に書き換えることができます。

<!--
In editions prior to 2021, raw lifetimes are parsed as separate tokens. For example `'r#foo` is parsed as three tokens: `'r`, `#`, and `foo`.
-->

2021 以前では、生ライフタイムは独立したトークンとみなされます。
たとえば、`'r#foo` は `'r`, `#`, `foo` の3トークンとして読み込まれます。

<!--
[raw identifiers]: ../../reference/identifiers.html#raw-identifiers
-->

[生識別子]: https://doc.rust-lang.org/reference/identifiers.html#raw-identifiers

<!--
## Migration
-->

## 移行

<!--
As a part of the 2021 edition a migration lint, [`rust_2021_prefixes_incompatible_syntax`], has been added in order to aid in automatic migration of Rust 2018 codebases to Rust 2021.
-->

Rust 2018 のコードベースから Rust 2021 への自動移行の支援のため、2021 エディションには、移行用のリント[`rust_2021_prefixes_incompatible_syntax`] が追加されています。

<!--
In order to migrate your code to be Rust 2021 Edition compatible, run:
-->

コードを Rust 2021 エディション互換に移行するには、次のように実行します。

```sh
cargo fix --edition
```

<!--
Should you want or need to manually migrate your code, migration is fairly straight-forward.
-->

手動でコードを移行する場合も、移行は比較的単純です。

<!--
Let's say you have a macro that is defined like so:
-->

たとえば、以下のようなマクロがあるとします。

```rust
macro_rules! my_macro {
    ($a:tt $b:tt $c:tt) => {};
}
```

<!--
In Rust 2015 and 2018 it's legal for this macro to be called like so with no space between the tokens:
-->

Rust 2015 と 2018 では、空白を入れずにトークンをつなげてマクロを呼び出しても合法です。

```rust,ignore
my_macro!('r#foo);
```

2021 エディションでは、これは一つのトークンとして解釈されます。
このマクロを呼び出すためには、以下のように識別子の前に空白を入れてください。

```rust,ignore
my_macro!('r# foo);
```

[`rust_2021_prefixes_incompatible_syntax`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#rust-2021-prefixes-incompatible-syntax
