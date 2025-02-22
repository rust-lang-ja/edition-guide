<!--
# Advanced migration strategies
-->

# 発展的な移行戦略

<!--
## How migrations work
-->

## 移行ツールの仕組み

<!--
[`cargo fix --edition`][`cargo fix`] works by running the equivalent of [`cargo check`] on your project with special [lints] enabled which will detect code that may not compile in the next edition.
These lints include instructions on how to modify the code to make it compatible on both the current and the next edition.
`cargo fix` applies these changes to the source code, and then runs `cargo check` again to verify that the fixes work.
If the fixes fail, then it will back out the changes and display a warning.
-->

[`cargo fix --edition`][`cargo fix`] コマンドは、[`cargo check`] コマンドと同様のコマンドを、次のエディションでコンパイルされなくなるコードを検知する特別な[リント]が有効になった状態で実行することで機能します。
このリントには、コードを変更したら現在と次のエディションの双方に適合させるための指示も含まれています。
`cargo fix` コマンドはソースコードをそれに従って変更し、再び `cargo check` を実行して修正がうまく行ったか確認します。
うまく行かなかった場合、変更を巻き戻して警告を表示します。

<!--
Changing the code to be simultaneously compatible with both the current and next edition makes it easier to incrementally migrate the code.
If the automated migration does not completely succeed, or requires manual help, you can iterate while staying on the original edition before changing `Cargo.toml` to use the next edition.
-->

現在と次のエディションの両方に同時に適合したコードに書き換えると、コードを段階的に移行することが楽になります。
自動移行が完全には成功しなかったか、手作業で変えてやる必要がある場合は、元ののエディション留まったまま同じことを繰り返してから、`Cargo.toml` を編集して次のエディションに進めてもよいです。

<!--
The lints that `cargo fix --edition` apply are part of a [lint group].
For example, when migrating from 2018 to 2021, Cargo uses the `rust-2021-compatibility` group of lints to fix the code.
Check the [Partial migration](#partial-migration-with-broken-code) section below for tips on using individual lints to help with migration.
-->

`cargo fix --edition` が適用するリントは、[リントグループ]の一部です。
例えば、2018 から 2021 に移行する場合、Cargo は `rust-2021-compatibility` というリントグループをコードの修正に使用します。
それぞれのリントを移行に役立てるコツについては、後の「[部分的な移行](#壊れたコードを元に部分的に移行する)」の章をご覧ください。

<!--
`cargo fix` may run `cargo check` multiple times.
For example, after applying one set of fixes, this may trigger new warnings which require further fixes.
Cargo repeats this until no new warnings are generated.
-->

`cargo fix` は、`cargo check` を複数回実行する可能性があります。
たとえば、一通りの修正を終えても、修正によって新たな警告が出て、さらなる修正が必要になるかもしれません。
Cargo は、新しく警告が出なくなるまでこれを繰り返し続けます。

<!--
## Migrating multiple configurations
-->

## 複数の設定を移行する

<!--
`cargo fix` can only work with a single configuration at a time.
If you use [Cargo features] or [conditional compilation], then you may need to run `cargo fix` multiple times with different flags.
-->

`cargo fix` は一度に1つのコンフィギュレーションでしか動きません。
[Cargo のフィーチャ]や[条件付きコンパイル]を使用している場合、`cargo fix` を異なるフラグで複数回実行する必要があるかもしれません。

<!--
For example, if you have code that uses `#[cfg]` attributes to include different code for different platforms, you may need to run `cargo fix` with the `--target` option to fix for different targets.
This may require moving your code between machines if you don't have cross-compiling available.
-->

例えば、あなたのコードが `#[cfg]` を使ってプラットフォームによって違うコードを含むようになっていた場合、`cargo fix` を `--target` オプションつきで実行して、ターゲットごとに修正をする必要があるかもしれません。
クロスコンパイルができない場合、コードを別のマシンに移して作業せざるを得ないかもしれません。

<!--
Similarly, if you have conditions on Cargo features, like `#[cfg(feature = "my-optional-thing")]`, it is recommended to use the `--all-features` flag to allow `cargo fix` to migrate all the code behind those feature gates.
If you want to migrate feature code individually, you can use the `--features` flag to migrate one at a time.
-->

同様に、フィーチャによる条件分岐、例えば `#[cfg(feature = "my-optional-thing")]` のようなものがある場合、`--all-features` フラグを使って、フィーチャの壁を超えて `cargo fix` がすべてのコードを変更できるようにするとよいでしょう。
フィーチャごとに別々にコードを移行したい場合は、`--features` フラグを使って一つずつ移行作業をすることもできます。

<!--
## Migrating a large project or workspace
-->

## 巨大なプロジェクトやワークスペースの移行

<!--
You can migrate a large project incrementally to make the process easier if you run into problems.
-->

大きなプロジェクトで問題が発生した場合、作業を単純にするために、段階的に移行していくこともできます。

<!--
In a [Cargo workspace], each package defines its own edition, so the process naturally involves migrating one package at a time.
-->

[Cargo のワークスペース]では、エディションはパッケージごとに定義されているため、自然とパッケージごとに1つずつ移行をすることになります。

<!--
Within a [Cargo package], you can either migrate the entire package at once, or migrate individual [Cargo targets] one at a time.
For example, if you have multiple binaries, tests, and examples, you can use specific target selection flags with `cargo fix --edition` to migrate just that one target.
By default, `cargo fix` uses `--all-targets`.
-->

[Cargo のパッケージ]においては、全パッケージを同時に移行することも、[Cargo ターゲット]ごとに1つずつ移行することもできます。
例えば、パッケージに複数のバイナリ、テスト、サンプルコード（`example` ターゲット）がある場合、`cargo fix --edition` コマンドに特定のターゲット選択用のフラグを組み合わせることで、一つのターゲットだけを移行することもできます。
デフォルトでは、`cargo fix` は `--all-targets` が暗黙に指定されていると扱います。

<!--
For even more advanced cases, you can specify the edition for each individual target in `Cargo.toml` like this:
-->

より発展的には、`Cargo.toml` 中に各ターゲットで使用するエディションを指定することもできます:

```toml
[[bin]]
name = "my-binary"
edition = "2018"
```

<!--
This usually should not be required, but is an option if you have a lot of targets and are having difficulty migrating them all together.
-->

おそらく普通はこれは必要ありませんが、ターゲットがたくさんあって全部いっぺんに移行作業できないような場合には一つの選択肢になるでしょう。

<!--
## Partial migration with broken code
-->

## 壊れたコードを元に部分的に移行する

<!--
Sometimes the fixes suggested by the compiler may fail to work.
When this happens, Cargo will report a warning indicating what happened and what the error was.
However, by default it will automatically back out the changes it made.
It can be helpful to keep the code in the broken state and manually resolve the issue.
Some of the fixes may have been correct, and the broken fix may be *mostly* correct, but just need minor tweaking.
-->

ときどき、コンパイラに提案された修正ではうまくいかないことがあります。
すると、Cargo は何が起こったかとどんなエラーが出たかを示す警告を報告しますが、デフォルトでは Cargo は変更を巻き戻します。
しかし、Cargo にはコードを壊れたままにしておいてもらい、手作業で問題を解決する、というのも有効な手段です。
ほとんどの修正は正しいかもしれませんし、壊れた修正も*だいたい*正しくて、少しいじれば問題ないかもしれないのです。

<!--
In this situation, use the `--broken-code` option with `cargo fix` to tell Cargo not to back out the changes.
Then, you can go manually inspect the error and investigate what is needed to fix it.
-->

そんなときは、`cargo fix` コマンドを実行するときに `--broken-code` オプションをつけて、Cargo が変更を巻き戻さないようにできます。
そうすれば、エラーを実際に見てみることも、修正点を確認することもできます。

<!--
Another option to incrementally migrate a project is to apply individual fixes separately, one at a time.
You can do this by adding the individual lints as warnings, and then either running `cargo fix` (without the `--edition` flag) or using your editor or IDE to apply its suggestions if it supports "Quick Fixes".
-->

プロジェクトを段階的に移行するもう一つの方法は、それぞれの修正を一つずつ適用することです。
このためには、各リントを警告として追加して、（`--edition` なしの）`cargo fix` を実行するか、お手持ちのエディタや IDE が「クイックフィックス」をサポートしていればそれを使って提案を適用すればよいです。

<!--
For example, the 2018 edition uses the [`keyword-idents`] lint to fix any conflicting keywords.
You can add `#![warn(keyword_idents)]` to the top of each crate (like at the top of `src/lib.rs` or `src/main.rs`).
Then, running `cargo fix` will apply just the suggestions for that lint.
-->

例えば、2018 エディションには [`keyword-idents`] という、キーワードとの衝突をすべて修正するためのリントがあります。
各クレートのトップ（例えば `src/lib.rs` や `src/main.rs` の先頭）に `#![warn(keyword_idents)]` を追加して、`cargo fix` を実行すれば、そのリントによる提案だけを受け入れることができます。

<!--
You can see the list of lints enabled for each edition in the [lint group] page, or run the `rustc -Whelp` command.
-->

各エディションで有効化されるリントの一覧は、[リントグループ]のページを見るか、 `rustc -Whelp` コマンドを実行すれば確認できます。

<!--
## Migrating macros
-->

## マクロの移行

<!--
Some macros may require manual work to fix them for the next edition.
For example, `cargo fix --edition` may not be able to automatically fix a macro that generates syntax that does not work in the next edition.
-->

マクロの中には、エディションを進めるにあたって手作業が必要なものがあります。
例えば、`cargo fix --edition` は、次のエディションで動作しない文法を生成するマクロを自動修正することは難しいかもしれません。

<!--
This may be a problem for both [proc macros] and `macro_rules`-style macros.
`macro_rules` macros can sometimes be automatically updated if the macro is used within the same crate, but there are several situations where it cannot.
Proc macros in general cannot be automatically fixed at all.
-->

これは、[手続き型マクロ]と `macro_rules` を使ったマクロの双方で問題になります。
`macro_rules` を使ったマクロは、マクロが同じクレートに属していたら自動でアップデートできる場合もありますが、いくつかの状況ではできません。
手続き型マクロは原則、全く修正できないと言っていいでしょう。

<!--
For example, if we migrate a crate containing this (contrived) macro `foo` from 2015 to 2018, `foo` would not be automatically fixed.
-->

例えば、この（わざとらしい）マクロ `foo` を含むクレートを 2015 から 2018 に移行しようとしても、`foo` は自動修復されません。

```rust
#[macro_export]
macro_rules! foo {
    () => {
        let dyn = 1;
        println!("it is {}", dyn);
              // "これは {} です"
    };
}
```

<!--
When this macro is defined in a 2015 crate, it can be used from a crate of any other edition due to macro hygiene (discussed below).
In 2015, `dyn` is a normal identifier and can be used without restriction.
-->

マクロが 2015 のクレートで定義されている場合、マクロの衛生性（後述）のおかげでこれは他のエディションのクレートからも使用することができます。
2015 では、`dyn` は通常の識別子で、制限なく使用できます。

<!--
However, in 2018, `dyn` is no longer a valid identifier.
When using `cargo fix --edition` to migrate to 2018, Cargo won't display any warnings or errors at all.
However, `foo` won't work when called from any crate.
-->

一方で、2018 では、`dyn` はもはや正当な識別子ではありません。
`cargo fix --edition` で 2018 へ移行するとき、Cargo は警告やエラーを一切表示しません。
しかし、`foo` はどのクレートで呼び出されても動作しません。

<!--
If you have macros, you are encouraged to make sure you have tests that fully cover the macro's syntax.
You may also want to test the macros by importing and using them in crates from multiple editions, just to ensure it works correctly everywhere.
If you run into issues, you'll need to read through the chapters of this guide to understand how the code can be changed to work across all editions.
-->

あなたのコードにマクロがある場合、そのマクロの構文を十分に網羅するテストがあることが推奨されます。
また、そのマクロを複数のエディションのクレートの中でインポートして使用し、どこでも動くということを確認するのも良いでしょう。
問題に突き当たった場合、このガイドの本章に目を通して、全エディションで動作するようにするためにどうすればいいか理解する必要があります。

<!--
### Macro hygiene
-->

### マクロの衛生性

<!--
Macros use a system called "edition hygiene" where the tokens within a macro are marked with which edition they come from.
This allows external macros to be called from crates of varying editions without needing to worry about which edition it is called from.
-->

マクロには、「エディション衛生性」と呼ばれる仕組みが使われています。このシステムでは、マクロ内のトークンがどのエディションから来たかが記録されています。
これにより、呼び出す側のエディションがどれであるかを気にせずに、外部のマクロを呼び出すことができます。

<!--
Let's take a closer look at the example above that defines a `macro_rules` macro using `dyn` as an identifier.
If that macro was defined in a crate using the 2015 edition, then that macro works fine, even if it were called from a 2018 crate where `dyn` is a keyword and that would normally be a syntax error.
The `let dyn = 1;` tokens are marked as being from 2015, and the compiler will remember that wherever that code gets expanded.
The parser looks at the edition of the tokens to know how to interpret it.
-->

上で例に出した、`macro_rules` で定義されたマクロを詳しく見てみましょう。
このマクロは `dyn` を識別子に使用しています。
このマクロが定義されているのが 2015 エディションのクレート内なのならば、これは問題なく動きます。
さらに、2018 エディションでは `dyn` はキーワードで普通は識別子に使えませんが、2018 エディションのクレートからこのマクロを呼び出したとしても大丈夫です。
パーサーはトークンのエディションに着目して、どう解釈したらよいかを判断します。

<!--
The problem arises when changing the edition to 2018 in the crate where it is defined.
Now, those tokens are tagged with the 2018 edition, and those will fail to parse.
However, since we never called the macro from our crate, `cargo fix --edition` never had a chance to inspect the macro and fix it.
-->

問題が起こるのは、マクロが定義されている側のクレートのエディションを 2018 に変更したときです。
今や、これらのトークンは 2018 エディションのものとしてタグ付けされているために、パースに失敗します。
しかしながら、このマクロは自身のクレートから呼び出されてもいないために、`cargo fix --edition` はこれを検査することも修正することもできません。

<!--
--> <!-- 原文にはここに TODO コメントがあります。左側の不自然なコメントは、 mdbook-transcheck で差分が出ないようにするためです -->

<!--
## Documentation tests
-->

## ドキュメンテーションテスト

<!--
At this time, `cargo fix` is not able to update [documentation tests].
After updating the edition in `Cargo.toml`, you should run `cargo test` to ensure everything still passes.
If your documentation tests use syntax that is not supported in the new edition, you will need to update them manually.
-->

現在のところ、`cargo fix` は[ドキュメンテーションテスト]を更新することはできません。
`Cargo.toml` でエディションを更新したら、`cargo test` を実行して全てのテストに通過することを確認すべきです。
新しいエディションでサポートされない構文がドキュメンテーションテストに使われていた場合は、手作業で修正する必要があります。

<!--
In rare cases, you can manually set the edition for each test.
For example, you can use the [`edition2018` annotation][rustdoc-annotation] on the triple backticks to tell `rustdoc` which edition to use.
-->

まれなケースですが、エディションをテストごとに設定することもできます。
例えば、バッククォート3つの後に [`edition2018` アノテーション][rustdoc-annotation]をつければ、`rustdoc` が使うエディションを指定できます。

<!--
## Generated code
-->

## 生成されるコード

<!--
Another area where the automated fixes cannot apply is if you have a build script which generates Rust code at compile time (see [Code generation] for an example).
In this situation, if you end up with code that doesn't work in the next edition, you will need to manually change the build script to generate code that is compatible.
-->

自動移行が使えない場所がもう一つあります。それは、ビルドスクリプトが Rust コードをコンパイル時に生成する場合です（具体例は「[コード生成]」をご参照ください）。
このとき、出てくるコードが次のエディションで機能しない場合は、生成されるコードが新しいエディションに適合するように、ビルドスクリプトを自分で書き換える必要があります。

<!--
## Migrating non-Cargo projects
-->

## Cargo でないプロジェクトの移行

<!--
If your project is not using Cargo as a build system, it may still be possible to make use of the automated lints to assist migrating to the next edition.
You can enable the migration lints as described above by enabling the appropriate [lint group].
For example, you can use the `#![warn(rust_2021_compatibility)]` attribute or the `-Wrust-2021-compatibility` or `--force-warns=rust-2021-compatibility` [CLI flag].
-->

プロジェクトのビルドシステムが Cargo 以外の場合でも、次のエディションへの移行に自動リントが利用できるかもしれません。
適切な[リントグループ]を使って、前述の移行リントを有効化できます。
例えば、`#![warn(rust_2021_compatibility)]` というアトリビュートを使ったり、`-Wrust-2021-compatibility` や `--force-warns=rust-2021-compatibility` などの [CLI フラグ]を使用するとよいです。

<!--
The next step is to apply those lints to your code.
There are several options here:
-->

次に、これらのリントをコードに適用します。
これにはいくつかの手があります:

<!--
* Manually read the warnings and apply the suggestions recommended by the compiler.
* Use an editor or IDE that supports automatically applying suggestions.
  For example, [Visual Studio Code] with the [Rust Analyzer extension] has the ability to use the "Quick Fix" links to automatically apply suggestions.
  Many other editors and IDEs have similar functionality.
* Write a migration tool using the [`rustfix`] library.
  This is the library that Cargo uses internally to take the [JSON messages] from the compiler and modify the source code.
  Check the [`examples` directory][rustfix-examples] for examples of how to use the library.
-->

* 警告を読んで、コンパイラに提案された変更を自分で加える。
* エディタや IDE の機能で、コンパイラからの提案を適用する。
  例えば、[Visual Studio Code] で [Rust Analyzer 拡張] を使えば、自動的に提案を受け入れるための「クイックフィックス」が使えます。
  他にも多くのエディタで同様の機能が使えます。
* [`rustfix`] ライブラリを用いて、移行ツールを自作する。
  このライブラリは Cargo 内部でも使われており、コンパイラからの [JSON メッセージ]を元にソースコードを編集します。
  ライブラリの使用例は、[`examples` ディレクトリ][rustfix-examples]をご覧ください。

<!--
## Writing idiomatic code in a new edition
-->

## 新しいエディションで慣用的な書き方をする

<!--
Editions are not only about new features and removing old ones.
In any programming language, idioms change over time, and Rust is no exception.
While old code will continue to compile, it might be written with different idioms today.
-->

エディションは、新機能の追加や古い機能の削除のためだけのものではありません。
どんなプログラミング言語でも、その言語らしい書き方は時代によって変化します。Rust も同じです。
古いプログラムはコンパイルには通るかもしれませんが、今は別の書き方があるかもしれません。

<!--
For example, in Rust 2015, external crates must be listed with `extern crate` like this:
-->

例えば、Rust 2015 では、外部クレートは以下のように `extern crate` で明示的に宣言される必要がありました:

```rust,ignore
// src/lib.rs
extern crate rand;
```

<!--
In Rust 2018, it is [no longer necessary](../rust-2018/path-changes.md#no-more-extern-crate) to include these items.
-->

Rust 2018 では、外部クレートを使うのに[`extern crate` は必要ありません](../rust-2018/path-changes.md#no-more-extern-crate)。

<!--
`cargo fix` has the `--edition-idioms` option to automatically transition some of these idioms to the new syntax.
-->

`cargo fix` には `--edition-idioms` オプションがあり、古い<ruby>書き方<rp>(</rp><rt>イディオム</rt><rp>)</rp></ruby>の一部を新しい書き方に書き換えることができます。

<!--
> **Warning**: The current *"idiom lints"* are known to have some problems.
> They may make incorrect suggestions which may fail to compile.
> The current lints are:
> * Edition 2018:
>     * [`unused-extern-crates`]
>     * [`explicit-outlives-requirements`]
> * Edition 2021 does not have any idiom lints.
>
> The following instructions are recommended only for the intrepid who are willing to work through a few compiler/Cargo bugs!
> If you run into problems, you can try the `--broken-code` option [described above](#partial-migration-with-broken-code) to make as much progress as possible, and then resolve the remaining issues manually.
-->

> **警告**: 現行の<!-- -->*「イディオムリント」*<!-- -->にはいくつか問題があることが知られています。
> これらのリントはときどき、受け入れるとコンパイルできなくなるような誤った提案をすることがあります。
> 現在、以下のリントがあります。
> * Edition 2018:
>     * [`unused-extern-crates`]
>     * [`explicit-outlives-requirements`]
> * Edition 2021 にイディオムリントはありません。
>
> 以下の手順は、コンパイラや Cargo の多少のバグを厭わない恐れ知らずだけにしかお勧めできません！　
> 不都合が発生する場合は、[前述](#壊れたコードを元に部分的に移行する)の `--broken-code` オプションを使ってツールにやれるだけのことをさせ、残った問題を自分の手で解決してもよいでしょう。

<!--
With that out of the way, we can instruct Cargo to fix our code snippet with:
-->

ともあれ、先程の短いコードを Cargo に直してもらうにはこうすればよいです:

```console
cargo fix --edition-idioms
```

<!--
Afterwards, the line with `extern crate rand;` in `src/lib.rs` will be removed.
-->

すると、`src/lib.rs` に書かれた `extern crate rand;` が削除されます。

<!--
We're now more idiomatic, and we didn't have to fix our code manually!
-->

これで、自分でコードに手を下すことなく、コードを現代風にできました！

<!--
[`cargo check`]: ../../cargo/commands/cargo-check.html
[`cargo fix`]: ../../cargo/commands/cargo-fix.html
[`explicit-outlives-requirements`]:  ../../rustc/lints/listing/allowed-by-default.html#explicit-outlives-requirements
[`keyword-idents`]: ../../rustc/lints/listing/allowed-by-default.html#keyword-idents
[`rustfix`]: https://crates.io/crates/rustfix
[`unused-extern-crates`]: ../../rustc/lints/listing/allowed-by-default.html#unused-extern-crates
[Cargo features]: ../../cargo/reference/features.html
[Cargo package]: ../../cargo/reference/manifest.html#the-package-section
[Cargo targets]: ../../cargo/reference/cargo-targets.html
[Cargo workspace]: ../../cargo/reference/workspaces.html
[CLI flag]: ../../rustc/lints/levels.html#via-compiler-flag
[Code generation]: ../../cargo/reference/build-script-examples.html#code-generation
[conditional compilation]: ../../reference/conditional-compilation.html
[documentation tests]: ../../rustdoc/documentation-tests.html
[JSON messages]: ../../rustc/json.html
[lint group]: ../../rustc/lints/groups.html
[lints]: ../../rustc/lints/index.html
[proc macros]: ../../reference/procedural-macros.html
[Rust Analyzer extension]: https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer
[rustdoc-annotation]: ../../rustdoc/documentation-tests.html#attributes
[rustfix-examples]: https://github.com/rust-lang/cargo/tree/master/crates/rustfix/examples
[Visual Studio Code]: https://code.visualstudio.com/
-->
[`cargo check`]: https://doc.rust-lang.org/cargo/commands/cargo-check.html
[`cargo fix`]: https://doc.rust-lang.org/cargo/commands/cargo-fix.html
[`explicit-outlives-requirements`]:  https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#explicit-outlives-requirements
[`keyword-idents`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#keyword-idents
[`rustfix`]: https://crates.io/crates/rustfix
[`unused-extern-crates`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unused-extern-crates
[Cargo のフィーチャ]: https://doc.rust-lang.org/cargo/reference/features.html
[Cargo のパッケージ]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-package-section
[Cargo ターゲット]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html
[Cargo のワークスペース]: https://doc.rust-lang.org/cargo/reference/workspaces.html
[CLI フラグ]: https://doc.rust-lang.org/rustc/lints/levels.html#via-compiler-flag
[コード生成]: https://doc.rust-lang.org/cargo/reference/build-script-examples.html#code-generation
[条件付きコンパイル]: https://doc.rust-lang.org/reference/conditional-compilation.html
[ドキュメンテーションテスト]: https://doc.rust-lang.org/rustdoc/documentation-tests.html
[JSON メッセージ]: https://doc.rust-lang.org/rustc/json.html
[リントグループ]: https://doc.rust-lang.org/rustc/lints/groups.html
[リント]: https://doc.rust-lang.org/rustc/lints/index.html
[手続き型マクロ]: https://doc.rust-lang.org/reference/procedural-macros.html
[Rust Analyzer 拡張]: https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer
[rustdoc-annotation]: https://doc.rust-lang.org/rustdoc/documentation-tests.html#attributes
[rustfix-examples]: https://github.com/rust-lang/cargo/tree/master/crates/rustfix/examples
[Visual Studio Code]: https://code.visualstudio.com/
