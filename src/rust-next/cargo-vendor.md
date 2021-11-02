# cargo vendor

<!--
Initially added: ![Minimum Rust version: 1.37](https://img.shields.io/badge/Minimum%20Rust%20Version-1.37-brightgreen.svg)
-->
![Minimum Rust version: 1.37](https://img.shields.io/badge/Minimum%20Rust%20Version-1.37-brightgreen.svg)で最初に導入されました。

<!--
After being available [as a separate crate][vendor-crate] for years, the `cargo vendor` command is now integrated directly into Cargo. The command fetches all your project's dependencies unpacking them into the `vendor/` directory, and shows the configuration snippet required to use the vendored code during builds.
-->
長年、[別のクレート][vendor-crate]として提供されてきたのちに、`cargo vendor`コマンドがCargoに直接統合されました。
このコマンドはあなたのプロジェクトの依存関係を全て取ってきて、`vendor/`ディレクトリの下に展開します。
そして、ビルド時にベンダーコードを使うために必要なコンフィグレーションの断片表示します。

<!--
There are multiple cases where `cargo vendor` is already used in production: the Rust compiler `rustc` uses it to ship all its dependencies in release tarballs, and projects with monorepos use it to commit the dependencies' code in source control.
-->
`cargo vendor`が既に使われている例が幾つかあります。
`rustc`コンパイラは全ての依存ライブラリをリリースアーカイブに入れるのに使い、モノリポのプロジェクトは依存コードをソースコード管理ツールにコミットするのに使っています。

[vendor-crate]: https://crates.io/crates/cargo-vendor
