> **Rust Edition Guide は現在 Rust 2024 のアップデート作業に向けて翻訳作業中です。本ページはある時点での英語版をコピーしていますが、一部のリンクが動作しない場合や、最新情報が更新されていない場合があります。問題が発生した場合は、[原文（英語版）](https://doc.rust-lang.org/nightly/edition-guide/introduction.html)をご参照ください。**

<!-- 
# Cargo: Rust-version aware resolver 
-->

# Cargo: Rustバージョンに基づいたリゾルバ

<!-- 
## Summary 
-->

## 概要

<!-- - `edition = "2024"` implies `resolver = "3"` in `Cargo.toml` which enables a Rust-version aware dependency resolver. -->

- `Cargo.toml` で `edition = "2024"` を指定すると、`resolver = "3"` が自動的に適用され、Rustバージョンに基づいた依存関係リゾルバが有効になります。

<!-- 
## Details 
-->

## 詳細

<!-- 
Since Rust 1.84.0, Cargo has opt-in support for compatibility with
[`package.rust-version`] to be considered when selecting dependency versions
by setting [`resolver.incompatible-rust-version = "fallback"`] in `.cargo/config.toml`. 
-->

Rust 1.84.0 以降、Cargo では [`package.rust-version`] を考慮した依存関係のバージョン選択を行うオプションが導入されました。
これを有効にするには、`.cargo/config.toml` に [`resolver.incompatible-rust-version = "fallback"`] の設定を追加します：

<!-- 
Starting in Rust 2024, this will be the default.
That is, writing `edition = "2024"` in `Cargo.toml` will imply `resolver = "3"`
which will imply [`resolver.incompatible-rust-version = "fallback"`]. 
-->

Rust 2024 Edition から、この設定がデフォルトになります。
つまり、`Cargo.toml` で `edition = "2024"` を指定すると、自動的に `resolver = "3"` が適用され、 [`resolver.incompatible-rust-version = "fallback"`] が有効になります。

<!-- 
The resolver is a global setting for a [workspace], and the setting is ignored in dependencies.
The setting is only honored for the top-level package of the workspace.
If you are using a [virtual workspace], you will still need to explicitly set the [`resolver` field]
in the `[workspace]` definition if you want to opt-in to the new resolver. 
-->

リゾルバは [workspace] に適用されるグローバルな設定であり、依存関係では無視されます。
この設定は workspace のトップレベルパッケージにのみ適用されます。
[virtual workspace] を使用している場合、新しいリゾルバを有効にするには [workspace] 定義内で [`resolver` フィールド] を明示的に設定する必要があります。

<!-- 
For more details on how Rust-version aware dependency resolution works, see [the Cargo book](../../cargo/reference/resolver.html#rust-version). 
-->

Rustバージョン対応の依存関係解決の詳細については、[the Cargo book](https://doc.rust-lang.org/cargo/reference/resolver.html#rust-version) を参照してください。

[`package.rust-version`]: https://doc.rust-lang.org/cargo/reference/rust-version.html
[`resolver.incompatible-rust-version = "fallback"`]: https://doc.rust-lang.org/cargo/reference/config.html#resolverincompatible-rust-versions
[workspace]: https://doc.rust-lang.org/cargo/reference/workspaces.html
[virtual workspace]: https://doc.rust-lang.org/cargo/reference/workspaces.html#virtual-workspace

<!-- 
[`resolver` field]: https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions 
-->

[`resolver` フィールド]: https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions 

<!-- 
## Migration 
-->

## 移行

<!-- 
There are no automated migration tools for updating for the new resolver. 
-->

新しいリゾルバへの更新を 自動で行うツールはありません。

<!-- 
We recommend projects
[verify against the latest dependencies in CI](../../cargo/guide/continuous-integration.html#verifying-latest-dependencies)
to catch bugs in dependencies as soon as possible. 
-->

プロジェクトでは、[CI で最新の依存関係を検証する](https://doc.rust-lang.org/cargo/guide/continuous-integration.html#verifying-latest-dependencies) ことを推奨します。
これにより、依存クレートのバグを早期に発見し適切に対応できます。


