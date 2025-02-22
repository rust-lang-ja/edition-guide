<!--
# Transitioning an existing project to a new edition
-->

# 既存のプロジェクトのエディションを移行する

<!--
Rust includes tooling to automatically transition a project from one edition to the next.
It will update your source code so that it is compatible with the next edition.
Briefly, the steps to update to the next edition are:
-->

Rust には、プロジェクトのエディションを更新するための自動移行ツールが付属しています。
このツールは、ソースコードを書き換えて次のエディションに適合させます。
新エディションを採用するために必要な手順は、概ね以下の通りです：

<!--
1. Run `cargo update` to update your dependencies to the latest versions.
2. Run `cargo fix --edition`
3. Edit `Cargo.toml` and set the `edition` field to the next edition, for example `edition = "2024"`
4. Run `cargo build` or `cargo test` to verify the fixes worked.
5. Run `cargo fmt` to reformat your project.
-->

1. `cargo update` を実行して、依存ライブラリを最新版にする
2. `cargo fix --edition` を実行する
3. `Cargo.toml` の `edition` フィールドを新しいエディションに設定する。たとえば、 `edition = "2024"` とする
4. `cargo build` や `cargo test` を実行して、修正がうまくいったことを検証する
5. `cargo fmt` を実行して、コードをリフォーマットする

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
> Zulip を使用しており、[こちら](https://rust-lang-jp.zulipchat.com/)から登録できます。

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
## Updating your dependencies
-->

## 依存ライブラリを更新する

<!--
Before we get started, it is recommended to update your dependencies. Some dependencies, particularly some proc-macros or dependencies that do build-time code generation, may have compatibility issues with newer editions. New releases may have been made since you last updated which may fix these issues. Run the following:
-->

作業前に、依存ライブラリを更新することをお勧めします。特に、proc macro や、ビルド時にコードを生成するような依存ライブラリは、エディションを進めることで互換性の問題が生じる場合があります。
依存ライブラリを更新をしていなかった間に、それらの問題を解消した新バージョンが出ているかもしれません。
以下のコマンドで依存ライブラリを更新できます。

```console
cargo update
```

<!--
After updating, you may want to run your tests to verify everything is working. If you are using a source control tool such as `git`, you may want to commit these changes separately to keep a logical separation of commits.
-->

更新後、コードが壊れていないことを確認するため、テストを実行しておくとよいでしょう。
`git` などのバージョン管理システムを使っているなら、適切なコミット粒度のためにここで一旦コミットしておくとよいでしょう。

<!--
## Updating your code to be compatible with the new edition
-->

## コードが新しいエディションでコンパイルできるようにする

<!--
Your code may or may not use features that are incompatible with the new edition.
In order to help transition to the next edition, Cargo includes the [`cargo fix`] subcommand to automatically update your source code.
To start, let's run it:
-->

コードは、新しいエディションと非互換な機能を使っている可能性があります。
Cargo の [`cargo fix`] サブコマンドで、コードを自動修正して次のエディションへ移行につなげることができます。
まず初めに、これを実行してみましょう。

```console
cargo fix --edition
```

<!--
This will check your code, and automatically fix any issues that it can.
Let's look at `src/lib.rs` again:
-->

コードは検査され、移行で発生する問題が自動的に修正されます。
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
この場合は、パラメータ名がなかったので、未使用パラメータの慣習に従って `_` を付加しています。

<!--
`cargo fix` can't always fix your code automatically.
If `cargo fix` can't fix something, it will print the warning that it cannot fix
to the console. If you see one of these warnings, you'll have to update your code manually.
See the [Advanced migration strategies] chapter for more on working with the migration process, and read the chapters in this guide which explain which changes are needed.
If you have problems, please seek help at the [user's forums](https://users.rust-lang.org/).
-->

`cargo fix`は常に自動的にコードを修正してくれるわけではありません。
`cargo fix` がコードを修正できない場合、コンソールに修正できなかったという警告を表示します。
そのときは手動でコードを修正してください。
「[発展的な移行戦略]」の節には、移行に関する詳細情報があります。
また他の節では、どのような変更が必要かについても説明していますので、併せてご参照ください。
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
## Testing your code in the new edition
-->

## 新エディションでテストを実行する

<!--
The next step is to test your project on the new edition.
Run your project tests to verify that everything still works, such as running [`cargo test`].
If new warnings are issued, you may want to consider running `cargo fix` again (without the `--edition` flag) to apply any suggestions given by the compiler.
-->

次に、新しいエディション上でプロジェクトのテストを実行しましょう。
[`cargo test`] を実行するなどして、プロジェクトのテストを走らせ、すべてが元のまま動くことを確認してください。
新たに警告が出た場合、(`--edition` なしの) `cargo fix` をもう一度実行することで、コンパイラからの提案を受け入れてみるのも良いかもしれません。

<!--
At this point, you may still need to do some manual changes. For example, the automatic migration does not update doctests, and build-time code generation or macros may need manual updating. See the [advanced migrations chapter] for more information.
-->

ここでも手動での変更が必要となる場合があります。たとえば、自動移行では doctest は修正されませんし、ビルド時コード生成やマクロなどは自分で書き換える必要があるかもしれません。
詳しくは「[発展的な移行戦略]」の節をご参照ください。

<!--
Congrats! Your code is now valid in both Rust 2015 and Rust 2018!
-->

わーい。今やあなたのコードは Rust 2015 と Rust 2018 の両方で有効です！

<!--
[advanced migrations chapter]: advanced-migrations.md
-->

<!--
## Reformatting with rustfmt
-->

## Rustfmt を用いてフォーマットする

<!--
If you use [rustfmt] to automatically maintain formatting within your project, then you should consider reformatting using the new formatting rules of the new edition.
-->

コードのフォーマットに [rustfmt] を利用しているなら、新エディションの新しいフォーマット規則に従ってフォーマットを実行すべきでしょう。

<!--
Before reformatting, if you are using a source control tool such as `git`, you may want to commit all the changes you have made up to this point before taking this step. It can be useful to put formatting changes in a separate commit, because then you can see which changes are just formatting versus other code changes, and also possibly ignore the formatting changes in `git blame`.
-->

`git` などのバージョン管理システムを使っているなら、フォーマット実行前に現時点で一旦コミットしておくとよいでしょう。
フォーマットの変更を別のコミットに分けておくと、コード整形とコードの書き換えを見分けたり、`git blame` でフォーマットの変更を無視したりと、後々役に立つかもしれません。

```console
cargo fmt
```

<!--
See the [style editions chapter] for more information.
-->

詳細は、[rustfmt の変更] に関する節をご参照ください。

<!--
[rustfmt]: https://github.com/rust-lang/rustfmt
[style editions chapter]: ../rust-2024/rustfmt-style-edition.md
-->

[rustfmt]: https://github.com/rust-lang/rustfmt
[rustfmt の変更]: ../rust-2024/rustfmt-style-edition.md

<!--
## Migrating to an unstable edition
-->

## 安定化されていないエディションに移行する

<!--
After an edition is released, there is roughly a three year window before the next edition.
During that window, new features may be added to the next edition, which will only be available on the [nightly channel].
If you want to help test those new features before they are stabilized, you can use the nightly channel to try them out.
-->

エディションのリリース後、次のエディションが出るまでざっと3年かかります。
それまでに、新機能が次期エディションに追加されることがあります。このエディションは [nightly チャンネル]限定です。
新機能の安定化前の試用に協力したい場合、nightly チャンネルを使うとよいです。

<!--
The steps are roughly similar to the stable channel:
-->

基本的には手順は stable チャンネルと同様です。

<!--
1. Install the most recent nightly: `rustup update nightly`.
2. Run `cargo +nightly fix --edition`.
3. Edit `Cargo.toml` and place `cargo-features = ["edition20xx"]` at the top (above `[package]`), and change the edition field to say `edition = "20xx"` where `20xx` is the edition you are upgrading to.
4. Run `cargo +nightly check` to verify it now works in the new edition.
-->

1. `rustup update nightly` で最新の nightly をインストールします。
2. `cargo +nightly fix --edition` を実行します。
3. `Cargo.toml` を編集して、`cargo-features = ["edition20xx"]` を最上部に（`[package]` より前に）記述して、`edition = "20xx"` のように edition フィールドを書き換えます（`20xx` は移行先のエディション）。
4. `cargo +nightly check` を実行して、新エディションでも問題ないことを確認します。

<!--
> **⚠ Caution**: Features implemented in the next edition may not have automatic migrations implemented with `cargo fix`, and the features themselves may not be finished.
> When possible, this guide should contain information about which features are implemented
> on nightly along with more information about their status.
> A few months before the edition is stabilized, all of the new features should be fully implemented, and the [Rust Blog] will announce a call for testing.
-->

> **⚠ 注意**: 次期エディションに実装された機能は、ものによっては `cargo fix` による自動移行が使えなかったり、機能自体が未完成だったりします。
> 本ガイドでは、nightly に実装されている機能の十分な情報や開発状況を可能な限り説明します。
> エディションが安定化される数ヶ月前までには、全ての新機能が完全に実装され、[Rust Blog] で試用が呼びかけられます。

<!--
[`cargo fix`]: ../../cargo/commands/cargo-fix.html
[`cargo test`]: ../../cargo/commands/cargo-test.html
[Advanced migration strategies]: advanced-migrations.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[Rust Blog]: https://blog.rust-lang.org/
-->

[`cargo fix`]: ../../cargo/commands/cargo-fix.html
[`cargo test`]: ../../cargo/commands/cargo-test.html
[発展的な移行戦略]: advanced-migrations.md
[nightly チャンネル]: ../../book/appendix-07-nightly-rust.html
[Rust Blog]: https://blog.rust-lang.org/
