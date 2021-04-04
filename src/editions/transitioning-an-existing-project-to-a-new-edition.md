<!--
# Transitioning an existing project to a new edition
-->
# 既存のプロジェクトのエディションを移行する

<!--
New editions might change the way you write Rust – they add new syntax,
language, and library features, and also remove features. For example, `try`,
`async`, and `await` are keywords in Rust 2018, but not Rust 2015. If you
have a project that's using Rust 2015, and you'd like to use Rust 2018 for it
instead, there's a few steps that you need to take.
-->

新たなエディションによってRustの書き方が変わるかも知れません。
新しい構文や新たなライブラリ機能の追加、そして時に機能の削除もあります。
例えば、`try`、`async`、`await`は Rust 2018ではキーワードですが、Rust 2015ではそうではありません。
もしあなたが Rust 2015のプロジェクトを持っていて、それを Rust 2018に移行したい場合には、やらなければならないことが幾つかあります。

<!--
> It's our intention that the migration to new editions is as smooth an
> experience as possible. If it's difficult for you to upgrade to Rust 2018,
> we consider that a bug. If you run into problems with this process, please
> [file a bug](https://github.com/rust-lang/rust/issues/new). Thank you!
-->

> 我々は、新しいエディションへの移行をできるだけスムーズに行えるようにしたいと考えています。
> もし、Rust 2018へアップグレードするのが大変な場合は、我々はそれをバグとみなします。
> もし移行時に問題があった場合には[バグ登録](https://github.com/rust-lang/rust/issues/new)してください。
>
> 訳注：Rustの日本語コミュニティもあります。
> Slackを使用しており[こちら](https://rust-jp.herokuapp.com/)から登録できます。

<!--
Here's an example. Imagine we have a crate that has this code in
`src/lib.rs`:
-->

ここに例を挙げます。`src/lib.rs`に以下のコードがあるクレートがあるとします。

```rust
trait Foo {
    fn foo(&self, Box<Foo>);
}
```

<!--
This code uses an anonymous parameter, that `Box<Foo>`. This is [not
supported in Rust 2018](../rust-2018/trait-system/no-anon-params.md), and
so this would fail to compile. Let's get this code up to date!
-->

このコードは `Box<Foo>`という無名パラメータを使用しています。
これは [Rust 2018ではサポートされておらず](../rust-2018/trait-system/no-anon-params.md)、コンパイルに失敗します。
このコードを更新してみましょう。

<!--
## Updating your code to be compatible with the new edition
-->

## あなたのコードを新しいエディションでコンパイルできるようにする

<!--
Your code may or may not use features that are incompatible with the new
edition. In order to help transition to Rust 2018, we've included a new
subcommand with Cargo. To start, let's run it:
-->

あなたのコードは互換性のない機能を使っているかも知れないし、使っていないかも知れません。
Rust 2018への移行を助けるためにCargoに新しいサブコマンドを追加しました。
まず初めにそれを起動してみましょう。

```console
> cargo fix --edition
```

<!--
This will check your code, and automatically fix any issues that it can.
Let's look at `src/lib.rs` again:
-->

これはあなたのコードをチェックして、自動的に移行の問題を修正してくれます。
もう一度 `src/lib.rs`を見てみましょう。

```rust
trait Foo {
    fn foo(&self, _: Box<Foo>);
}
```

<!--
It's re-written our code to introduce a parameter name for that trait object.
In this case, since it had no name, `cargo fix` will replace it with `_`,
which is conventional for unused variables.
-->

トレイトオブジェクトのためのパラメータ名が追加された形でコードが書き換えられています。
この場合は、パラメータ名がなかったので、使用されていないパラメータの慣習に従って `_` を付加しています。

<!--
`cargo fix` can't always fix your code automatically.
If `cargo fix` can't fix something, it will print the warning that it cannot fix
to the console. If you see one of these warnings, you'll have to update your code
manually. See the corresponding section of this guide for help, and if you have
problems, please seek help at the [user's forums](https://users.rust-lang.org/).
-->

`Cargo fix`は常に自動的にコードを修正してくれるわけではありません。
もし、`cargo fix`がコードを修正できない時にはコンソールに修正できなかったという警告を表示します。
その場合は手動でコードを修正してください。
助けが必要な時は、このガイドの対応するセクションを参照してください。
問題がある場合は、 [ユーザーフォーラム](https://users.rust-lang.org/)で助けを求めてください。

<!--
Keep running `cargo fix --edition` until you have no more warnings.
-->

そして警告が出なくなるまで `cargo fix --edition` を繰り返し実行してください。

<!--
Congrats! Your code is now valid in both Rust 2015 and Rust 2018!
-->

おめでとうございます！ あなたのコードはRust 2015とRust 2018の双方で正しいコードになりました。

<!--
## Enabling the new edition to use new features
-->

## 新機能を使うために新たなエディションを有効化する

<!--
In order to use some new features, you must explicitly opt in to the new
edition. Once you're ready to commit, change your `Cargo.toml` to add the new
`edition` key/value pair. For example:
-->

新しいエディションの新機能を使うには明示的にオプトインする必要があります。
コミットする準備ができたら、`Cargo.toml`に新しいエディションのキーバリューペアを追加してください。
例えば以下のような形になります。


```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"
```

<!--
If there's no `edition` key, Cargo will default to Rust 2015. But in this case,
we've chosen `2018`, and so our code is compiling with Rust 2018!
-->

もし `edition`キーがなければCargoはデフォルトで Rust 2015をエディションとして使います。
しかし上記の例では、`2018`を明示的に指定しているのでコードは Rust 2018でビルドされます。

<!--
## Writing idiomatic code in a new edition
-->

## 新しいエディションで慣用的なコードを書く

<!--
Editions are not only about new features and removing old ones. In any programming
language, idioms change over time, and Rust is no exception. While old code
will continue to compile, it might be written with different idioms today.
-->

エディションは新機能を追加したり機能を削除するだけのものではありません。
どのようなプログラミング言語でも、イディオム（プログラムの書き方のスタイル）は時と共に変化していきます。
Rustも例外ではありません。
古いスタイルのコードは引き続きコンパイル可能ですが、新しいエディションでは違った書き方で書いた方が良いかも知れません。

<!--
Our sample code contains an outdated idiom. Here it is again:
-->

我々のサンプルコードは古いスタイルを含んでいます。
もう一度ここにそのコードを示します。

```rust
trait Foo {
    fn foo(&self, _: Box<Foo>);
}
```

<!--
In Rust 2018, it's considered idiomatic to use the [`dyn`
keyword](../rust-2018/trait-system/dyn-trait-for-trait-objects.md) for
trait objects.
-->

Rust 2018では、トレイトオブジェクトに [`dyn` キーワード](../rust-2018/trait-system/dyn-trait-for-trait-objects.md) を付けるのが良いとされています。

<!--
Eventually, we want `cargo fix` to fix all these idioms automatically in the same
manner we did for upgrading to the 2018 edition. **Currently,
though, the *"idiom lints"* are not ready for widespread automatic fixing.** The
compiler isn't making `cargo fix`-compatible suggestions in many cases right
now, and it is making incorrect suggestions in others. Enabling the idiom lints,
even with `cargo fix`, is likely to leave your crate either broken or with many
warnings still remaining.
-->

いずれ、`cargo fix`によってこのようなイディオムの変更も、2018エディションへアップグレードしたときのように自動的に行いたいと考えています。
**ただし今現在は、イディオムチェッカーが広範囲の自動修正をできるレベルにはなっていません。**
今のところ、コンパイラは多くの場合 `cargo fix`互換のサジェスチョンを出さなかったり、間違ったサジェスチョンを出したりします。
 `cargo fix`と共にイディオムチェッカーを有効にすると、おそらくはあなたのコードを壊してしまったり、多くの警告が残り続けるということになってしまいます。

<!--
We have plans to make these idiom migrations a seamless part of the Rust 2018
experience, but we're not there yet. As a result the following instructions are
recommended only for the intrepid who are willing to work through a few
compiler/Cargo bugs!
-->

Rust 2018の体験の一部として、シームレスなイディオム移行を提供する計画があります。
しかしまだそこには至っていません。
したがって、以下の手順はコンパイラやCargoのバグを乗り越えることを厭わない勇猛な方のみにお勧めします。


<!--
With that out of the way, we can instruct Cargo to fix our code snippet with:
-->

以上を踏まえたうえで、私たちのコード片をCargoに修正させてみましょう。

```console
$ cargo fix --edition-idioms
```

<!--
Afterwards, `src/lib.rs` looks like this:
-->

実行後は `src/lib.rs`は以下のようになります。


```rust
trait Foo {
    fn foo(&self, _: Box<dyn Foo>);
}
```

<!--
We're now more idiomatic, and we didn't have to fix our code manually!
-->

これでコードはより新しいスタイルになりました。
手修正する必要はありませんでした。

<!--
Note that `cargo fix` may still not be able to automatically update our code.
If `cargo fix` can't fix something, it will print a warning to the console, and
you'll have to fix it manually.
-->

なお、`cargo fix`はコードを自動的に改修することができない場合もあることを覚えておいてください。
その場合は `cargo fix`は警告メッセージを出すので、それを見て手動でコードを修正してください。

<!--
As mentioned before, there are known bugs around the idiom lints which
means they're not all ready for prime time yet. You may get a scary-looking
warning to report a bug to Cargo, which happens whenever a fix proposed by
`rustc` actually caused code to stop compiling by accident. If you'd like `cargo
fix` to make as much progress as possible, even if it causes code to stop
compiling, you can execute:
-->

上でも述べたようにイディオムチェッカーには幾つかわかっているバグがあり、まだ実践登用できるレベルではありません。
Cargoのバグレポートを出すようにという恐ろしげな警告を見るかも知れませんが、これは `rustc`によって提案された修正が誤ってコードをコンパイルできなくしてしまった時に起こります。
もしコンパイルが止まったとしても `cargo fix`を使ってできるだけ自動修正をしたい場合には以下のコマンドを使います。

```console
$ cargo fix --edition-idioms --broken-code
```

<!--
This will instruct `cargo fix` to apply automatic suggestions regardless of
whether they work or not. Like usual, you'll see the compilation result after
all fixes are applied. If you notice anything wrong or unusual, please feel free
to report an issue to Cargo and we'll help prioritize and fix it.
-->

これは、動くかどうかは関係なく`cargo fix`に自動修正を行わせます。
全ての修正が適用された後にコードはコンパイルされてその結果を見ることができます。
もし何か間違いや異常に気がついた時は、お気軽にCargoにバグ報告してください。

<!--
Enjoy the new edition!
-->

それでは、新しいエディションをお楽しみください！
