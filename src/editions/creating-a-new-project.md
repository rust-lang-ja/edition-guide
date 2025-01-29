<!--
# Creating a new project

A new project created with Cargo is configured to use the latest edition by
default:
-->

# 新しいプロジェクトを作成する

Cargo で新たなプロジェクトを作成すると、最新のエディションを使う設定が自動的に記述されます。

```console
$ cargo new foo
    Creating binary (application) `foo` package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
$ cat foo/Cargo.toml
[package]
name = "foo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

<!--
That `edition = "2024"` setting configures your package to be built using the
Rust 2024 edition. No further configuration needed!

You can use the `--edition <YEAR>` option of `cargo new` to create the project
using some specific edition. For example, creating a new project to use the
Rust 2018 edition could be done like this:
-->

`edition = "2024"` で、パッケージが Rust 2024 でビルドされるよう設定されます。
これ以外は必要ありません。

`cargo new` のオプションとして `--edition <YEAR>` を用いれば、
新しいプロジェクトがそのエディションを使うようにできます。
たとえば、Rust 2018 を使ったプロジェクトを作るには、以下のようにします。

```console
$ cargo new --edition 2018 foo
    Creating binary (application) `foo` package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
$ cat foo/Cargo.toml
[package]
name = "foo"
version = "0.1.0"
edition = "2018"

[dependencies]
```

<!--
Don't worry about accidentally using an invalid year for the edition; the
`cargo new` invocation will not accept an invalid edition year value:
-->

うっかり存在しないエディションを指定しても大丈夫です。
`cargo new` に存在しないエディションを指定しても弾かれます。

```console
$ cargo new --edition 2019 foo
error: invalid value '2019' for '--edition <YEAR>'
  [possible values: 2015, 2018, 2021, 2024]

  tip: a similar value exists: '2021'

For more information, try '--help'.
```

<!--
You can change the value of the `edition` key by simply editing the
`Cargo.toml` file. For example, to cause your package to be built using the
Rust 2015 edition, you would set the key as in the following example:
-->

`Cargo.toml` の `edition` の値を直接変更しても構いません。
たとえば、パッケージが Rust 2015 でビルドされるようにするには、以下のように設定すればよいです。

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["your name <you@example.com>"]
edition = "2015"

[dependencies]
```
