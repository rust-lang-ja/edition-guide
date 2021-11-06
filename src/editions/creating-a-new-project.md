<!--
# Creating a new project

When you create a new project with Cargo, it will automatically add
configuration for the latest edition:
-->

# 新しいプロジェクトを作成する

Cargoは新たなプロジェクトを作成する際に自動で最新のエディションをコンフィギュレーションに追加します。

```console
> cargo +nightly new foo
     Created binary (application) `foo` project
> cat foo/Cargo.toml
[package]
name = "foo"
version = "0.1.0"
authors = ["your name <you@example.com>"]
edition = "2021"

[dependencies]
```

<!--
That `edition = "2021"` setting will configure your package to use Rust 2021.
No more configuration needed!

If you'd prefer to use an older edition, you can change the value in that
key, for example:
-->

この `edition = "2021"` によってあなたのパッケージが Rust 2021 を利用するように設定されます。
これ以外は必要ありません。

もし、他の古いエディションを使いたい場合は、その設定の値を変更できます。例えば、

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["your name <you@example.com>"]
edition = "2015"

[dependencies]
```

<!--
This will build your package in Rust 2015.
-->

とすると、あなたのパッケージは Rust 2015 でビルドされます。
