<!--
# Transitioning an existing project to a new edition
-->
# 既存のプロジェクトのエディションを移行する

<!--
Rust includes tooling to automatically transition a project from one edition to the next.
It will update your source code so that it is compatible with the next edition.
Briefly, the steps to update to the next edition are:
-->

Rust には、プロジェクトのエディションを進めるための自動移行ツールが付属しています。
このツールは、あなたのソースコードを書き換えて次のエディションに適合させます。
簡単にいうと、新しいエディションに進むためには次のようにすればよいです。

<!--
1. Run `cargo fix --edition`
2. Edit `Cargo.toml` and set the `edition` field to the next edition, for example `edition = "2021"`
3. Run `cargo build` or `cargo test` to verify the fixes worked.
-->

1. `cargo fix --edition` を実行する
2. `Cargo.toml` の `edition` フィールドを新しいエディションに設定する。たとえば、 `edition = "2021"` とする
3. `cargo build` や `cargo test` を実行して、修正がうまくいったことを検証する。

<!--
The following sections dig into the details of these steps, and some of the issues you may encounter along the way.
-->

以下のセクションで、これらの手順の詳細と、その途中で起こりうる問題点について詳しく説明します。

<!--
> It's our intention that the migration to new editions is as smooth an
> experience as possible. If it's difficult for you to upgrade to the latest edition,
> we consider that a bug. If you run into problems with this process, please
> [file a bug](https://github.com/rust-lang/rust/issues/new/choose). Thank you!
-->

> 我々は、新しいエディションへの移行をできるだけスムーズに行えるようにしたいと考えています。
> もし、最新のエディションにアップグレードするのが大変な場合は、我々はそれをバグとみなします。
> もし移行時に問題があった場合には[バグ登録](https://github.com/rust-lang/rust/issues/new/choose)してください。
> よろしくお願いします！
>
> 訳注：Rustの日本語コミュニティもあります。
> Slackを使用しており[こちら](https://rust-jp.herokuapp.com/)から登録できます。

<!--
## Starting the migration
-->

## 移行の開始

<!--
As an example, let's take a look at transitioning from the 2015 edition to the 2018 edition.
The steps are essentially the same when transitioning to other editions like 2021.
-->

例えば、2015エディションから2018エディションに移行する場合を見てみましょう。
ここで説明する手順は、例えば2021エディションのように、別のエディションに移行する場合も実質的に同様です。

<!--
Imagine we have a crate that has this code in `src/lib.rs`:
-->

`src/lib.rs`に以下のコードがあるクレートがあるとします。

```rust
trait Foo {
    fn foo(&self, i32);
}
```

<!--
This code uses an anonymous parameter, that `i32`. This is [not
supported in Rust 2018](../rust-2018/trait-system/no-anon-params.md), and
so this would fail to compile. Let's get this code up to date!
-->

このコードは `i32` という無名パラメータを使用しています。
これは [Rust 2018ではサポートされておらず](../rust-2018/trait-system/no-anon-params.md)、コンパイルに失敗します。
このコードを更新してみましょう。

<!--
## Updating your code to be compatible with the new edition
-->

## あなたのコードを新しいエディションでコンパイルできるようにする

<!--
Your code may or may not use features that are incompatible with the new edition.
In order to help transition to the next edition, Cargo includes the [`cargo fix`] subcommand to automatically update your source code.
To start, let's run it:
-->

あなたのコードは新しいエディションに互換性のない機能を使っているかもしれないし、使っていないかもしれません。
Cargo には [`cargo fix`] というサブコマンドがあり、これがあなたのコードを自動的に更新して次のエディションへの移行を補助してくれます。
まず初めに、これを実行してみましょう。

```console
cargo fix --edition
```

<!--
This will check your code, and automatically fix any issues that it can.
Let's look at `src/lib.rs` again:
-->

これはあなたのコードをチェックして、自動的に移行の問題を修正してくれます。
もう一度 `src/lib.rs`を見てみましょう。

```rust
trait Foo {
    fn foo(&self, _: i32);
}
```

<!--
It's re-written our code to introduce a parameter name for that `i32` value.
In this case, since it had no name, `cargo fix` will replace it with `_`,
which is conventional for unused variables.
-->

`i32` 値をとるパラメータに名前が追加された形でコードが書き換えられています。
この場合は、パラメータ名がなかったので、使用されていないパラメータの慣習に従って `_` を付加しています。

<!--
`cargo fix` can't always fix your code automatically.
If `cargo fix` can't fix something, it will print the warning that it cannot fix
to the console. If you see one of these warnings, you'll have to update your code manually.
See the [Advanced migration strategies] chapter for more on working with the migration process, and read the chapters in this guide which explain which changes are needed.
If you have problems, please seek help at the [user's forums](https://users.rust-lang.org/).
-->

`cargo fix`は常に自動的にコードを修正してくれるわけではありません。
もし、`cargo fix`がコードを修正できない時にはコンソールに修正できなかったという警告を表示します。
その場合は手動でコードを修正してください。
「[発展的な移行戦略]」<!-- TODO: 章の名前に合わせてリンク名を変える必要があるかもしれません -->の章では、移行に関するより多くの情報があります。また、このガイドの他の章では、どのような変更が必要かについても説明しますので、併せてご参照ください。
問題が発生したときは、[ユーザーフォーラム](https://users.rust-lang.org/) で助けを求めてください。

<!--
## Enabling the new edition to use new features
-->

## 新機能を使うために新たなエディションを有効化する

<!--
In order to use some new features, you must explicitly opt in to the new
edition. Once you're ready to continue, change your `Cargo.toml` to add the new
`edition` key/value pair. For example:
-->

新しいエディションの新機能を使うには明示的にオプトインする必要があります。
準備がよければ、`Cargo.toml` に新しい `edition` のキーバリューペアを追加してください。
例えば以下のような形になります。


```toml
[package]
name = "foo"
version = "0.1.0"
edition = "2018"
```

<!--
If there's no `edition` key, Cargo will default to Rust 2015. But in this case,
we've chosen `2018`, and so our code will compile with Rust 2018!
-->

もし `edition` キーがなければCargoはデフォルトで Rust 2015をエディションとして使います。
しかし、上記の例では `2018` を明示的に指定しているので、コードは Rust 2018 でコンパイルされます！

<!--
The next step is to test your project on the new edition.
Run your project tests to verify that everything still works, such as running [`cargo test`].
If new warnings are issued, you may want to consider running `cargo fix` again (without the `--edition` flag) to apply any suggestions given by the compiler.
-->

次に、新しいエディション上であなたのプロジェクトをテストしましょう。
[`cargo test`] を実行するなどして、プロジェクトのテストを走らせ、すべてが元のまま動くことを確認してください。
新たに警告が出た場合、(`--edition` なしの) `cargo fix` をもう一度実行することで、コンパイラからの提案を受け入れてみるのも良いかもしれません。

<!--
Congrats! Your code is now valid in both Rust 2015 and Rust 2018!
-->

わーい。今やあなたのコードは Rust 2015 と Rust 2018 の両方で有効です！

<!--
[`cargo fix`]: ../../cargo/commands/cargo-fix.html
[`cargo test`]: ../../cargo/commands/cargo-test.html
[Advanced migration strategies]: advanced-migrations.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
-->

[`cargo fix`]: ../../cargo/commands/cargo-fix.html
[`cargo test`]: ../../cargo/commands/cargo-test.html
[発展的な移行戦略]: advanced-migrations.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
